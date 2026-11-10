# 03-hybrid-retrieval

> M3 · LLM application và RAG — Khối 3 · Tuần 20–21 · Retrieval nâng cao

**Trạng thái:** ⬜ Chưa bắt đầu

## Mục tiêu

- Thêm BM25 có tách từ; hybrid bằng RRF
- Thêm reranker cho top-20 → lấy top-5
- Khi chunk được chọn dẫn chiếu tới điều khác, kéo thêm điều đó vào context
- Đo recall@5 trên 50 câu có nhãn cho từng cấu hình: chỉ vector, chỉ BM25, hybrid, hybrid + rerank

## Tiêu chí đạt

- [ ] có bảng recall@5 cho 4 cấu hình; viết ADR chọn cấu hình dựa trên số liệu, có tính cả độ trễ.

## Lý thuyết cần đọc

- [ ] BM25 và vì sao cần tách từ tiếng Việt (ví dụ thư viện underthesea hoặc pyvi)
- [ ] Hybrid search và Reciprocal Rank Fusion
- [ ] Reranker dạng cross-encoder: chậm hơn nhưng chính xác hơn
- [ ] Viết lại truy vấn: câu hỏi nối tiếp, từ viết tắt sinh viên hay dùng (“học lại”, “cảnh báo học vụ”)
- [ ] Dẫn chiếu chéo trong văn bản pháp quy: “theo quy định tại khoản 2 Điều 10”

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
