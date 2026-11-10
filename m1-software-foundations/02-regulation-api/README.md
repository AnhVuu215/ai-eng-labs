# 02-regulation-api

> M1 · Nền móng phần mềm — Khối 2 · Tuần 3–4 · FastAPI và PostgreSQL

**Trạng thái:** ⬜ Chưa bắt đầu

## Mục tiêu

- API CRUD cho văn bản và điều khoản, dữ liệu lấy từ doc-loader
- Endpoint tìm kiếm full-text; đo thời gian truy vấn trước và sau khi thêm index
- Endpoint /stream trả từng đoạn text qua SSE (giả lập LLM), client hiển thị dần

## Tiêu chí đạt

- [ ] migration chạy lại từ DB trống không lỗi; tìm kiếm có index nhanh hơn rõ rệt và bạn ghi lại số đo; test API chạy trên DB test riêng.

## Lý thuyết cần đọc

- [ ] Thiết kế REST, mã lỗi, phân trang; dependency injection trong FastAPI
- [ ] Streaming bằng Server-Sent Events (cách mọi ứng dụng chat LLM trả token)
- [ ] SQLAlchemy 2.0 + Alembic migration; index B-tree và GIN; transaction và mức cô lập
- [ ] Full-text search của Postgres (tsvector), nền móng cho BM25 ở M3

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
