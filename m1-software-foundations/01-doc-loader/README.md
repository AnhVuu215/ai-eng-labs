# 01-doc-loader

> M1 · Nền móng phần mềm — Khối 1 · Tuần 1–2 · Python production

**Trạng thái:** ⬜ Chưa bắt đầu

## Mục tiêu

- Viết thư viện đọc file PDF/DOCX quy chế đào tạo, xuất JSON giữ cấu trúc chương → điều → khoản
- Thêm lệnh CLI tải song song 20 URL với giới hạn 5 request đồng thời, timeout 10 giây, retry 3 lần
- Viết test cho các trường hợp: file hỏng, điều khoản lồng nhau, URL lỗi 500 và lỗi timeout

## Tiêu chí đạt

- [ ] kiểm tra kiểu strict không lỗi; độ phủ test ≥ 80%; tách đúng cấu trúc ít nhất 1 văn bản quy chế thật.

## Lý thuyết cần đọc

- [ ] Type hints, Protocol, generics; Pydantic v2 để validate dữ liệu vào/ra
- [ ] Quản lý dự án bằng uv, cấu trúc package src/, pyproject.toml
- [ ] asyncio: event loop, gather, Semaphore giới hạn đồng thời, timeout
- [ ] Retry có exponential backoff và jitter; khi nào không nên retry
- [ ] pytest: fixture, parametrize, mock gọi HTTP

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
