# Hướng dẫn: Redis, hàng đợi, độ tin cậy

> Thời lượng: 2 tuần · Làm tiếp trên code của [02-regulation-api](../02-regulation-api/)

## Cách tổ chức code

Lab này **mở rộng** regulation-api. Bạn có hai cách:
- Copy `02-regulation-api` sang thư mục này rồi sửa tiếp. Cách này giữ nguyên lab 02 để so sánh trước/sau, nên được khuyến nghị.
- Hoặc sửa thẳng lab 02 trên một branch git riêng.

## Bạn sẽ làm ra gì

```
POST /documents            → 202 Accepted {"job_id": "..."}   (không parse ngay nữa)
GET  /jobs/{job_id}        → {"status": "queued|running|done|failed", "result": ...}
Mọi endpoint               → giới hạn N request/phút/người dùng, vượt thì 429 + Retry-After
POST có Idempotency-Key    → gửi lại cùng key không tạo bản ghi thứ hai
GET  /search               → cache trong Redis, có đo tỷ lệ hit
```

Thêm vào `docker-compose.yml`:

```yaml
  redis:
    image: redis:7
    ports: ["6379:6379"]
  worker:
    build: .
    command: uv run arq regulation_api.worker.WorkerSettings
    depends_on: [db, redis]
```

```bash
uv add redis arq
```

Chọn **arq** vì nó async thuần, hợp với FastAPI và nhẹ. Celery mạnh hơn nhưng cấu hình nặng. Ghi lý do chọn vào README.

---

## Bước 1: Đưa ingest sang worker nền (4–5 giờ)

```python
# worker.py
async def ingest_document(ctx: dict, file_path: str, doc_meta: dict) -> int:
    doc = parse(extract_lines(Path(file_path)), source=file_path)
    async with SessionLocal() as s:
        ...  # lưu vào DB như cũ
    return document_id

class WorkerSettings:
    functions = [ingest_document]
    redis_settings = RedisSettings.from_dsn(settings.redis_url)
    max_tries = 3                 # retry tác vụ
    job_timeout = 300
```

```python
# routes/documents.py
@router.post("/documents", status_code=202)
async def create(file: UploadFile, pool: ArqDep) -> JobOut:
    path = save_upload(file)           # lưu file ra đĩa/volume trước
    job = await pool.enqueue_job("ingest_document", str(path), {...})
    return JobOut(job_id=job.job_id)

@router.get("/jobs/{job_id}")
async def job_status(job_id: str, pool: ArqDep) -> JobStatusOut:
    job = Job(job_id, pool)
    status = await job.status()        # queued / in_progress / complete / not_found
    ...
```

**Thí nghiệm bắt buộc (tiêu chí đạt):** worker chết giữa chừng thì tác vụ có chạy lại không?
1. Thêm `await asyncio.sleep(20)` vào giữa `ingest_document`.
2. Gửi request, đợi worker log "started", rồi `docker compose kill worker`.
3. `docker compose up -d worker` và quan sát. Ghi kết quả vào README.

Đọc tài liệu arq về `retry` và hành vi khi worker chết. Nếu tác vụ được chạy lại, nó có tạo **dữ liệu đôi** không? Nếu có, đây là lúc idempotency ở bước 3 trở nên cần thiết.

**Dead-letter:** tác vụ thất bại hết `max_tries` thì ghi vào bảng `failed_jobs` (job_id, lỗi, input, thời gian) để xem lại, không bỏ mất.

✅ **Checkpoint:** upload trả 202 ngay, `/jobs/{id}` chuyển dần sang `done`, thí nghiệm kill worker có kết quả ghi lại. **Commit.**

## Bước 2: Rate limit (3 giờ)

Cài **token bucket** hoặc **sliding window** bằng Redis. Phải dùng thao tác nguyên tử (Lua script hoặc `MULTI`), nếu không 2 request song song sẽ cùng đọc được "còn 1 lượt".

Sliding window đơn giản bằng sorted set:

```python
async def allow(redis, key: str, limit: int, window_s: int) -> tuple[bool, int]:
    now = time.time()
    async with redis.pipeline(transaction=True) as p:
        p.zremrangebyscore(key, 0, now - window_s)    # bỏ request cũ
        p.zadd(key, {f"{now}-{uuid4()}": now})
        p.zcard(key)
        p.expire(key, window_s)
        _, _, count, _ = await p.execute()
    allowed = count <= limit
    retry_after = ...   # tính từ phần tử cũ nhất còn trong cửa sổ
    return allowed, retry_after
```

Áp dụng bằng middleware hoặc dependency. Người dùng xác định theo header `X-User-Id` (tạm thời) hoặc IP.

```python
if not allowed:
    raise HTTPException(429, headers={"Retry-After": str(retry_after)})
```

Test: gửi `limit + 1` request thì request cuối nhận 429 và header `Retry-After` > 0.

Tự trả lời: đoạn code trên vẫn ghi request bị từ chối vào cửa sổ. Như vậy đúng hay sai? Hãy so với token bucket.

✅ **Checkpoint:** test 429 xanh. **Commit.**

## Bước 3: Idempotency key (3–4 giờ)

Luồng xử lý:

```
Client gửi POST + header Idempotency-Key: abc
  ├─ Redis SET idem:abc "processing" NX EX 86400
  │    ├─ set được  → xử lý → lưu response vào idem:abc → trả response
  │    └─ đã tồn tại → giá trị là response   → trả lại đúng response cũ
  │                  → giá trị "processing"  → 409 Conflict (request trước chưa xong)
```

Chi tiết quan trọng:
- Lưu kèm **hash của body**. Cùng key mà khác body thì trả 422, vì client đang dùng sai key.
- Xử lý lỗi (5xx) thì **xóa** key để client được retry.
- Ở tầng DB nên có thêm ràng buộc `UNIQUE` (ví dụ trên hash nội dung file) làm lớp bảo vệ thứ hai.

Test (tiêu chí đạt): gửi cùng request 2 lần (tuần tự và song song bằng `asyncio.gather`), sau đó `SELECT count(*)` phải bằng 1.

✅ **Checkpoint:** test idempotency xanh cả tuần tự và song song. **Commit.**

## Bước 4: Cache kết quả tìm kiếm (2–3 giờ)

Cache-aside:

```python
key = f"search:v1:{sha1(normalize(q))}:{limit}"
if (hit := await redis.get(key)) is not None:
    await redis.incr("stats:search:hit"); return json.loads(hit)
await redis.incr("stats:search:miss")
result = await repo.search(q, limit)
await redis.set(key, json.dumps(result), ex=600)
```

**Invalidation:** khi có văn bản mới hoặc văn bản bị sửa thì cache tìm kiếm bị cũ. Có 3 cách, chọn một và ghi lý do:
1. TTL ngắn, chấp nhận dữ liệu lệch tối đa N phút.
2. Đổi version trong key (`search:v2:`) mỗi khi ingest xong, các key cũ tự hết hạn.
3. Xóa theo pattern (`SCAN` + `DEL`). Cách này chậm khi có nhiều key; hãy tìm hiểu vì sao không dùng `KEYS`.

Đo tỷ lệ hit: viết script gửi 1.000 truy vấn theo phân phối thực tế (vài câu phổ biến lặp nhiều, nhiều câu hiếm). Gợi ý: lấy mẫu theo phân phối Zipf bằng `numpy.random.zipf`.

```
hit_rate = hit / (hit + miss)
```

✅ **Checkpoint:** có số hit rate và p95 khi hit so với khi miss. **Commit.**

## Kết quả đo cần ghi

| Chỉ số | Giá trị |
|---|---|
| Thời gian phản hồi POST /documents: trước (đồng bộ) và sau (202) | |
| Kill worker giữa chừng: tác vụ có chạy lại không, có dữ liệu đôi không | |
| Gửi trùng 2 lần (tuần tự, song song): số bản ghi | phải = 1 |
| Cache hit rate với 1.000 truy vấn phân phối Zipf | |
| p95 tìm kiếm: hit / miss | |

## Lỗi hay gặp

| Triệu chứng | Nguyên nhân |
|---|---|
| Worker không thấy job | API và worker dùng khác Redis DB/URL, hoặc khác tên hàm |
| Rate limit cho qua quá giới hạn khi gửi song song | đọc rồi mới ghi, không nguyên tử |
| Idempotency vẫn tạo 2 bản ghi khi gửi song song | dùng `GET` rồi `SET` thay vì `SET NX` |
| Cache trả dữ liệu cũ mãi | quên invalidation, TTL quá dài |

## Tài liệu

- Redis docs: data types (string, sorted set), transactions/pipelines, Lua scripting
- arq docs: jobs, retries
- Stripe Engineering: "Designing robust and predictable APIs with idempotency"
- DDIA chương 1 (reliability) và chương 11 (stream processing, phần exactly-once)
