# Hướng dẫn: doc-loader

> Thời lượng: 2 tuần · Yêu cầu trước: [docs/HUONG-DAN-CHUNG.md](../../docs/HUONG-DAN-CHUNG.md)

## Bạn sẽ làm ra gì

Một thư viện Python kèm lệnh CLI:

```bash
uv run doc-loader parse quy-che.pdf -o quy-che.json      # PDF/DOCX → JSON có cấu trúc
uv run doc-loader fetch urls.txt -o downloads/           # tải song song có giới hạn, timeout, retry
```

JSON đầu ra giữ cấu trúc **chương → điều → khoản**. Đây là nền cho mọi module sau: M1 lưu nó vào Postgres, M3 dùng mỗi điều/khoản làm một chunk RAG.

## Cấu trúc thư mục đề xuất

```
01-doc-loader/
├── pyproject.toml
├── src/doc_loader/
│   ├── __init__.py
│   ├── models.py       # Pydantic: Document, Chapter, Article, Clause
│   ├── extract.py      # PDF/DOCX → danh sách dòng text
│   ├── structure.py    # dòng text → cây chương/điều/khoản
│   ├── fetch.py        # tải async có Semaphore, timeout, retry
│   └── cli.py          # typer app
├── tests/
│   ├── fixtures/       # file mẫu: quy chế thật, file hỏng, file lồng nhau
│   ├── test_structure.py
│   └── test_fetch.py
└── README.md
```

---

## Bước 1: Khởi tạo dự án (30 phút)

```bash
cd m1-software-foundations/01-doc-loader
uv init --package --name doc_loader .
uv add pydantic typer pypdf python-docx httpx
uv add --dev pytest pytest-cov pytest-asyncio respx mypy ruff
```

Dán cấu hình ruff/mypy/pytest trong HUONG-DAN-CHUNG vào `pyproject.toml`. Thêm entry point CLI:

```toml
[project.scripts]
doc-loader = "doc_loader.cli:app"
```

✅ **Checkpoint:** `uv run mypy src` và `uv run pytest` chạy được (chưa có test cũng được). **Commit.**

## Bước 2: Mô hình dữ liệu bằng Pydantic (1 giờ)

Trước khi viết parser, chốt hình dạng đầu ra:

```python
# models.py
from datetime import date
from pydantic import BaseModel, Field

class Clause(BaseModel):
    number: int                      # khoản 1, 2, 3...
    text: str
    points: list[str] = Field(default_factory=list)   # điểm a), b)...

class Article(BaseModel):
    number: int                      # Điều 10
    title: str
    clauses: list[Clause] = Field(default_factory=list)
    intro: str = ""                  # đoạn text trước khoản 1 (nếu có)

class Chapter(BaseModel):
    number: str                      # "I", "II"... (số La Mã)
    title: str
    articles: list[Article] = Field(default_factory=list)

class Document(BaseModel):
    source: str
    title: str
    effective_date: date | None = None
    chapters: list[Chapter]
```

Tự trả lời trước khi đi tiếp: điều khoản **không thuộc chương nào** thì đặt ở đâu? Quy chế có "điểm a, b, c" trong khoản thì sao? Ghi lựa chọn của bạn vào README.

✅ **Checkpoint:** `Document.model_json_schema()` in ra schema đúng ý bạn.

## Bước 3: Trích text từ PDF và DOCX (2–3 giờ)

```python
# extract.py
from pathlib import Path

class ExtractError(Exception): ...

def extract_lines(path: Path) -> list[str]:
    match path.suffix.lower():
        case ".pdf":  return _from_pdf(path)
        case ".docx": return _from_docx(path)
        case _: raise ExtractError(f"Không hỗ trợ định dạng {path.suffix}")
```

Gợi ý:
- PDF: `pypdf.PdfReader(path).pages[i].extract_text()`. File hỏng sẽ ném `pypdf.errors.PdfReadError`, hãy bắt lỗi đó và ném lại thành `ExtractError` kèm tên file.
- DOCX: `docx.Document(path).paragraphs` → `p.text`.
- Chuẩn hóa: `unicodedata.normalize("NFC", s)`. PDF tiếng Việt hay trả về dạng tổ hợp (NFD), làm regex so khớp sai mà mắt không thấy.
- Bỏ dòng rỗng, số trang, header/footer lặp lại.

⚠️ **Bẫy hay gặp:** PDF scan (ảnh) không có text, `extract_text()` trả chuỗi rỗng. Hãy phát hiện trường hợp này và báo lỗi rõ ràng, đừng trả JSON rỗng một cách im lặng.

✅ **Checkpoint:** in 50 dòng đầu của một văn bản quy chế thật của HUCE, đọc được tiếng Việt có dấu. **Commit.**

## Bước 4: Tách cấu trúc chương → điều → khoản (4–6 giờ, phần chính)

Văn bản quy phạm Việt Nam có mẫu khá đều:

```
Chương I            → ^Chương\s+([IVXLC]+)
QUY ĐỊNH CHUNG      → dòng tiếp theo là tiêu đề chương
Điều 1. Phạm vi...  → ^Điều\s+(\d+)\.\s*(.*)
1. Quy chế này...   → ^(\d+)\.\s+(.*)      (khoản)
a) Sinh viên...     → ^([a-zđ])\)\s+(.*)    (điểm)
```

Cách làm: duyệt từng dòng như một **máy trạng thái**. Giữ `current_chapter`, `current_article`, `current_clause`. Gặp mẫu nào thì mở node mới; dòng không khớp mẫu nào thì nối vào node đang mở (vì một khoản có thể dài nhiều dòng).

```python
# structure.py (khung)
import re
CHAPTER = re.compile(r"^Chương\s+([IVXLC]+)\b", re.IGNORECASE)
ARTICLE = re.compile(r"^Điều\s+(\d+)\.?\s*(.*)")
CLAUSE  = re.compile(r"^(\d+)\.\s+(.*)")
POINT   = re.compile(r"^([a-zđ])\)\s+(.*)")

def parse(lines: list[str], source: str) -> Document:
    ...
```

⚠️ **Bẫy hay gặp:**
- Dòng "1. Quy chế này..." nằm **trước** "Điều 1" (ví dụ trong phần căn cứ) không được nhận nhầm thành khoản.
- Trong thân khoản có câu "theo quy định tại Điều 10" mà xuống dòng đúng chỗ đó, nên dòng mới bắt đầu bằng "Điều 10" và bị nhận nhầm thành điều mới. Cách kiểm tra: số điều phải tăng dần.
- Số khoản phải bắt đầu lại từ 1 ở mỗi điều mới.

Viết test **trước** khi sửa từng bẫy:

```python
@pytest.mark.parametrize("lines,expected_articles", [
    (["Chương I", "QUY ĐỊNH CHUNG", "Điều 1. Phạm vi", "1. Nội dung a"], 1),
    # thêm: khoản nhiều dòng, điểm a) b), điều không có khoản, "Điều 10" giữa câu...
])
def test_parse_counts(lines, expected_articles): ...
```

✅ **Checkpoint:** chạy trên 1 quy chế thật, mở JSON ra và **đối chiếu tay 5 điều ngẫu nhiên** với bản PDF. Ghi tỷ lệ đúng vào README. **Commit.**

## Bước 5: Tải song song có giới hạn, timeout, retry (3–4 giờ)

Yêu cầu: 20 URL, tối đa 5 request cùng lúc, timeout 10 giây, retry 3 lần.

```python
# fetch.py (khung)
import asyncio, random
import httpx

RETRYABLE = {429, 500, 502, 503, 504}

async def fetch_one(client: httpx.AsyncClient, url: str, sem: asyncio.Semaphore,
                    retries: int = 3) -> FetchResult:
    async with sem:                                 # giới hạn đồng thời
        for attempt in range(retries + 1):
            try:
                r = await client.get(url, timeout=10.0)
                if r.status_code in RETRYABLE: raise RetryableStatus(r.status_code)
                r.raise_for_status()
                return FetchResult(url=url, ok=True, content=r.content)
            except (httpx.TimeoutException, httpx.TransportError, RetryableStatus) as e:
                if attempt == retries:
                    return FetchResult(url=url, ok=False, error=repr(e))
                await asyncio.sleep(_backoff(attempt))
    ...

def _backoff(attempt: int, base: float = 0.5, cap: float = 8.0) -> float:
    return random.uniform(0, min(cap, base * 2 ** attempt))   # "full jitter"

async def fetch_all(urls: list[str], limit: int = 5) -> list[FetchResult]:
    sem = asyncio.Semaphore(limit)
    async with httpx.AsyncClient() as client:
        return await asyncio.gather(*(fetch_one(client, u, sem) for u in urls))
```

Tự trả lời và ghi vào notes:
- Vì sao **không** retry lỗi 404 hay 400?
- Vì sao cần jitter? Hãy hình dung 1.000 client cùng retry đúng sau 1 giây.
- `fetch_one` tự bắt lỗi và trả `FetchResult(ok=False)` thay vì ném lỗi. Nhờ vậy một URL hỏng không làm `gather` hủy cả 19 URL còn lại. Thử bỏ cách này đi để xem chuyện gì xảy ra.

Test bằng `respx` (giả lập httpx, không gọi mạng thật):

```python
@respx.mock
async def test_retry_then_success():
    route = respx.get("https://x/a").mock(side_effect=[httpx.Response(500), httpx.Response(200, content=b"ok")])
    [res] = await fetch_all(["https://x/a"])
    assert res.ok and route.call_count == 2
```

Cần test đủ: 500 rồi thành công · 500 liên tục (hết retry) · timeout (`side_effect=httpx.ReadTimeout`) · 404 (không retry) · đo được tối đa 5 request chạy cùng lúc.

Gợi ý cho test đếm đồng thời: tạo một mock handler `async` có `await asyncio.sleep(0.05)`, tăng/giảm một biến đếm toàn cục và ghi lại giá trị lớn nhất.

✅ **Checkpoint:** tất cả test fetch xanh, đo được max concurrency = 5. **Commit.**

## Bước 6: CLI bằng Typer (1 giờ)

```python
# cli.py
import typer
app = typer.Typer()

@app.command()
def parse(path: Path, out: Path = typer.Option(..., "-o")) -> None: ...

@app.command()
def fetch(urls_file: Path, out_dir: Path = typer.Option(..., "-o"), limit: int = 5) -> None:
    results = asyncio.run(fetch_all(urls, limit))
    # in bảng: url | ok | số lần thử | thời gian
```

✅ **Checkpoint:** hai lệnh trong phần "Bạn sẽ làm ra gì" chạy được từ terminal.

## Bước 7: Đạt tiêu chí (1–2 giờ)

```bash
uv run mypy src                 # strict, 0 lỗi
uv run pytest --cov=doc_loader  # ≥ 80%
```

Coverage thấp thì xem dòng nào chưa phủ (`--cov-report=term-missing`). Thường đó là nhánh xử lý lỗi, nên viết test cho file hỏng.

## Điền "Kết quả đo" trong README

| Chỉ số | Cách đo |
|---|---|
| mypy strict | số lỗi |
| Coverage | % từ pytest-cov |
| Độ chính xác tách cấu trúc | số điều đúng / số điều kiểm tra tay trên văn bản thật |
| Thời gian parse | `time` cho 1 văn bản, ghi số trang |
| Tải 20 URL | tổng thời gian với limit=1 so với limit=5 |

## Lỗi hay gặp

| Triệu chứng | Nguyên nhân thường gặp |
|---|---|
| Regex không khớp dù nhìn giống hệt | Unicode NFD và NFC khác nhau; khoảng trắng không ngắt (`\xa0`) |
| `mypy` báo lỗi với `pypdf`/`docx` | thư viện thiếu type stub: thêm `[[tool.mypy.overrides]] module = ["docx.*"] ignore_missing_imports = true` |
| Test async không chạy | thiếu `asyncio_mode = "auto"` trong cấu hình pytest |
| Cả `gather` sập vì 1 URL | để exception lọt ra khỏi `fetch_one` |

## Mở rộng (nếu còn thời gian)

- Trích `effective_date` từ câu "có hiệu lực từ ngày...".
- Ghi lại các dẫn chiếu chéo ("theo khoản 2 Điều 10") thành trường `references`, M3 khối 3 sẽ cần.
- Dùng `hypothesis` để sinh văn bản ngẫu nhiên và kiểm tra parser không bao giờ crash.

## Tài liệu

- Fluent Python, chương về type hints và asyncio
- [httpx async](https://www.python-httpx.org/async/) · [respx](https://lundberg.github.io/respx/) · [Typer](https://typer.tiangolo.com/)
- AWS Architecture Blog: "Exponential Backoff And Jitter"
