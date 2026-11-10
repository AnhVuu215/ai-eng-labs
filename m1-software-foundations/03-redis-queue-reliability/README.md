# 03-redis-queue-reliability

> M1 · Nền móng phần mềm — Khối 3 · Tuần 5–6 · Redis, hàng đợi, độ tin cậy

**Trạng thái:** ⬜ Chưa bắt đầu

## Mục tiêu

- Chuyển việc ingest văn bản sang worker chạy nền; API trả job_id, có endpoint xem trạng thái
- Thêm rate limit theo người dùng; trả 429 kèm Retry-After
- Thêm idempotency key cho endpoint tạo dữ liệu
- Cache kết quả tìm kiếm phổ biến; đo tỷ lệ cache hit

## Tiêu chí đạt

- [ ] gửi cùng một request hai lần chỉ tạo một bản ghi; worker bị tắt giữa chừng thì tác vụ được chạy lại; có số đo cache hit.

## Lý thuyết cần đọc

- [ ] Cache-aside, TTL, cache invalidation
- [ ] Rate limit (token bucket, sliding window)
- [ ] Hàng đợi tác vụ nền (arq, RQ hoặc Celery), retry tác vụ, dead-letter
- [ ] Idempotency key: vì sao gửi lại một request không được tạo dữ liệu đôi

## Cách chạy

```bash
# TODO
```

## Kết quả đo

| Chỉ số | Giá trị | Ghi chú |
|---|---|---|
|  |  |  |

## Ghi chép / bài học

- 
