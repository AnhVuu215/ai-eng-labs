# 02-rag-v1

> M3 · LLM application và RAG — Khối 2 · Tuần 18–19 · RAG v1 end-to-end

**Trạng thái:** ⬜ Chưa bắt đầu

## Mục tiêu

- Ingest bằng doc-loader từ M1; mỗi chunk là một điều hoặc khoản, metadata gồm văn bản, chương, điều, khoản, ngày hiệu lực
- Embed bằng model đa ngôn ngữ đã chọn ở M2, lưu vào pgvector
- Endpoint /ask: lấy top-5, sinh câu trả lời kèm trích dẫn dạng “Điều 10, khoản 2”; trả lời “không tìm thấy trong quy chế” khi không đủ căn cứ
- Giao diện chat đơn giản có streaming; đưa cho 5 người bạn dùng thử
- Thu 50 câu hỏi thật đầu tiên, tự gán điều khoản đúng cho từng câu

## Tiêu chí đạt

- [ ] chạy end-to-end trên URL công khai; có 50 câu hỏi thật có nhãn (hạt giống cho golden set ở M4).

## Lý thuyết cần đọc

- [ ] Kiến trúc RAG: ingest → chunk → embed → lưu → truy xuất → sinh câu trả lời
- [ ] Chunking: cố định độ dài, theo câu, theo cấu trúc văn bản; metadata đi kèm chunk
- [ ] pgvector: kiểu index (HNSW, IVFFlat), đánh đổi tốc độ và độ chính xác
- [ ] Prompt sinh câu trả lời có trích dẫn và hướng dẫn từ chối

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
