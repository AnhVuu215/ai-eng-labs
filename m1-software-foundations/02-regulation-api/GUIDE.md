# Hướng dẫn: regulation-api

> Thời lượng: 2 tuần · Cần xong trước: [01-doc-loader](../01-doc-loader/)

## Bạn sẽ làm ra gì

API FastAPI trên PostgreSQL:

```
POST   /documents                 nhận file → doc-loader → lưu văn bản + điều + khoản
GET    /documents                 danh sách, có phân trang
GET    /documents/{id}            chi tiết kèm cây điều khoản
GET    /articles/{id}
GET    /search?q=học+lại&limit=10 full-text search tiếng Việt
GET    /stream?q=...              SSE: trả text từng đoạn (giả lập LLM)
```

## Cấu trúc thư mục

```
02-regulation-api/
├── docker-compose.yml        # postgres (+ postgres-test)
├── alembic.ini
├── migrations/versions/
├── src/regulation_api/
│   ├── main.py               # tạo app, include router
│   ├── config.py             # pydantic-settings
│   ├── db.py                 # engine, session dependency
│   ├── models.py             # SQLAlchemy ORM
│   ├── schemas.py            # Pydantic vào/ra
│   ├── repository.py         # truy vấn DB (tách khỏi route)
│   └── routes/{documents,search,stream}.py
└── tests/
    ├── conftest.py           # DB test riêng, transaction rollback mỗi test
    └── test_*.py
```

---

## Bước 1: Postgres bằng Docker (30 phút)

```yaml
# docker-compose.yml
services:
  db:
    image: postgres:17
    environment: { POSTGRES_USER: app, POSTGRES_PASSWORD: app, POSTGRES_DB: app }
    ports: ["5432:5432"]
    volumes: [pgdata:/var/lib/postgresql/data]
  db-test:
    image: postgres:17
    environment: { POSTGRES_USER: app, POSTGRES_PASSWORD: app, POSTGRES_DB: app_test }
    ports: ["5433:5432"]
    tmpfs: [/var/lib/postgresql/data]       # dữ liệu test nằm trong RAM, tự xóa
volumes: { pgdata: {} }
```

```bash
docker compose up -d
uv init --package --name regulation_api .
uv add fastapi "uvicorn[standard]" "sqlalchemy[asyncio]" "psycopg[binary]" alembic pydantic-settings sse-starlette
uv add --editable ../01-doc-loader
uv add --dev pytest pytest-asyncio httpx mypy ruff
```

✅ **Checkpoint:** `docker compose ps` thấy 2 container healthy.

## Bước 2: Model và migration (3–4 giờ)

```python
# models.py (khung) — SQLAlchemy 2.0 typed style
class Base(DeclarativeBase): ...

class DocumentRow(Base):
    __tablename__ = "documents"
    id: Mapped[int] = mapped_column(primary_key=True)
    title: Mapped[str]
    source: Mapped[str]
    effective_date: Mapped[date | None]
    created_at: Mapped[datetime] = mapped_column(server_default=func.now())
    articles: Mapped[list["ArticleRow"]] = relationship(back_populates="document", cascade="all, delete-orphan")

class ArticleRow(Base):
    __tablename__ = "articles"
    id: Mapped[int] = mapped_column(primary_key=True)
    document_id: Mapped[int] = mapped_column(ForeignKey("documents.id", ondelete="CASCADE"), index=True)
    chapter: Mapped[str]
    number: Mapped[int]
    title: Mapped[str]
    # ClauseRow tương tự: article_id, number, text
```

Tự quyết định và ghi lý do vào README: lưu khoản thành **bảng riêng** hay **cột JSONB** trong `articles`? Nên nghĩ đến việc M3 cần mỗi khoản là một chunk có id riêng.

```bash
uv run alembic init -t async migrations
# sửa migrations/env.py: target_metadata = Base.metadata, URL đọc từ config
uv run alembic revision --autogenerate -m "init schema"
uv run alembic upgrade head
```

⚠️ Luôn **đọc file migration autogenerate sinh ra** trước khi chạy. Nó hay bỏ sót hoặc sinh sai index đặc biệt (GIN, cột generated).

✅ **Checkpoint:** chạy `alembic downgrade base && alembic upgrade head` không lỗi. Đây là tiêu chí "migration chạy lại từ DB trống". **Commit.**

## Bước 3: CRUD và phân trang (3–4 giờ)

Dependency injection cho session:

```python
# db.py
engine = create_async_engine(settings.database_url)
SessionLocal = async_sessionmaker(engine, expire_on_commit=False)

async def get_session() -> AsyncIterator[AsyncSession]:
    async with SessionLocal() as session:
        yield session

SessionDep = Annotated[AsyncSession, Depends(get_session)]
```

Phân trang: chọn **offset/limit** (đơn giản) hoặc **cursor/keyset** (`WHERE id > :last_id`, ổn định khi dữ liệu thay đổi). Hãy làm cả hai trên `/documents` rồi so sánh, đây là câu hỏi phỏng vấn hay gặp.

Mã lỗi: 404 khi không thấy, 422 khi dữ liệu sai (FastAPI tự trả), 409 khi trùng, 413 khi file quá lớn. Lỗi trả về cùng một dạng:

```json
{"error": {"code": "document_not_found", "message": "Không tìm thấy văn bản 42"}}
```

✅ **Checkpoint:** mở `http://localhost:8000/docs`, upload 1 file quy chế thật, xem được cây điều khoản. **Commit.**

## Bước 4: Full-text search và đo index (4 giờ)

Postgres không có cấu hình tiếng Việt sẵn, nên dùng `simple` (không stemming) cộng `unaccent` để tìm được cả khi gõ không dấu.

```sql
-- trong migration (op.execute)
CREATE EXTENSION IF NOT EXISTS unaccent;
-- unaccent() không IMMUTABLE nên không dùng trực tiếp trong cột generated → bọc lại:
CREATE FUNCTION f_unaccent(text) RETURNS text LANGUAGE sql IMMUTABLE PARALLEL SAFE
  AS $$ SELECT public.unaccent('public.unaccent', $1) $$;

ALTER TABLE clauses ADD COLUMN tsv tsvector
  GENERATED ALWAYS AS (to_tsvector('simple', f_unaccent(text))) STORED;
```

Truy vấn:

```sql
SELECT id, ts_rank(tsv, q) AS rank, ts_headline('simple', text, q) AS snippet
FROM clauses, websearch_to_tsquery('simple', f_unaccent(:q)) q
WHERE tsv @@ q ORDER BY rank DESC LIMIT :limit;
```

**Đo trước/sau index** (đây là tiêu chí đạt):
1. Sinh dữ liệu đủ lớn: nhân bản các khoản lên ≥ 100.000 dòng bằng `generate_series` hoặc script.
2. Chạy `EXPLAIN (ANALYZE, BUFFERS)` cho 1 truy vấn, chụp kết quả (sẽ thấy `Seq Scan`).
3. `CREATE INDEX ix_clauses_tsv ON clauses USING GIN (tsv);` (viết thành migration mới).
4. Chạy lại (sẽ thấy `Bitmap Index Scan`). Ghi thời gian cả hai lần.
5. Đo p50/p95 cho 100 truy vấn khác nhau, cả trước và sau.

✅ **Checkpoint:** bảng số đo trước/sau trong README, kèm số dòng dữ liệu. **Commit.**

## Bước 5: Streaming bằng SSE (2 giờ)

```python
# routes/stream.py
from sse_starlette.sse import EventSourceResponse

@router.get("/stream")
async def stream(q: str) -> EventSourceResponse:
    async def gen() -> AsyncIterator[dict[str, str]]:
        for word in FAKE_ANSWER.split():
            await asyncio.sleep(0.05)           # giả lập LLM sinh token
            yield {"event": "token", "data": word + " "}
        yield {"event": "done", "data": ""}
    return EventSourceResponse(gen())
```

Client: một file `static/index.html` dùng `new EventSource('/stream?q=...')`, nối dần text vào trang. Mở DevTools → Network để xem từng event đến.

Tự trả lời: client đóng tab giữa chừng thì generator có dừng không? Hãy thử bằng cách in log trong khối `finally`.

✅ **Checkpoint:** text hiện dần trên trình duyệt. **Commit.**

## Bước 6: Test trên DB test riêng (3 giờ)

Mỗi test chạy trong một transaction rồi rollback, nhanh và không bẩn dữ liệu:

```python
# conftest.py (khung)
TEST_URL = "postgresql+psycopg://app:app@localhost:5433/app_test"

@pytest.fixture(scope="session")
async def engine():
    eng = create_async_engine(TEST_URL)
    # chạy alembic upgrade head lên DB test (gọi alembic.command.upgrade)
    yield eng
    await eng.dispose()

@pytest.fixture
async def session(engine):
    async with engine.connect() as conn:
        trans = await conn.begin()
        s = AsyncSession(bind=conn, join_transaction_mode="create_savepoint")
        yield s
        await trans.rollback()

@pytest.fixture
async def client(session):
    app.dependency_overrides[get_session] = lambda: session
    async with httpx.AsyncClient(transport=httpx.ASGITransport(app=app), base_url="http://t") as c:
        yield c
    app.dependency_overrides.clear()
```

Test tối thiểu: tạo/đọc/xóa văn bản · 404 · phân trang (trang cuối, trang rỗng) · tìm có dấu và không dấu ra cùng kết quả · SSE nhận đủ event `done`.

✅ **Checkpoint:** `uv run pytest` xanh, không đụng DB dev. **Commit.**

## Kết quả đo cần ghi

| Chỉ số | Ghi chú |
|---|---|
| Số dòng `clauses` dùng để đo | |
| p50/p95 tìm kiếm: không index / có GIN | `EXPLAIN ANALYZE` + 100 truy vấn |
| Kích thước index GIN | `SELECT pg_size_pretty(pg_relation_size('ix_clauses_tsv'))` |
| Offset vs keyset ở trang 1000 | thời gian |
| Số test, coverage | |

## Lỗi hay gặp

| Triệu chứng | Nguyên nhân |
|---|---|
| `MissingGreenlet` | truy cập relationship lazy trong async. Dùng `selectinload()` |
| autogenerate không thấy bảng | `env.py` chưa import models nên `Base.metadata` rỗng |
| `functions in index expression must be marked IMMUTABLE` | dùng `unaccent` trực tiếp; cần hàm bọc `f_unaccent` |
| Test thứ 2 thấy dữ liệu test thứ 1 | thiếu rollback hoặc session bị commit thật |

## Tài liệu

- FastAPI: Dependencies, Bigger Applications, Testing
- SQLAlchemy 2.0: ORM Quickstart, AsyncIO extension
- PostgreSQL docs: Chapter 12 Full Text Search; Chapter 11 Indexes (GIN)
- Designing Data-Intensive Applications, chương 3 (index)
