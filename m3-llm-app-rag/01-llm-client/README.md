# 01-llm-client

> M3 · LLM application và RAG — Khối 1 · Tuần 16–17 · LLM API và context engineering

**Trạng thái:** ⬜ Chưa bắt đầu

## Mục tiêu

- Viết lớp LLMClient dùng chung cho 2 nhà cung cấp; streaming; retry; ghi log token, độ trễ, chi phí mỗi lần gọi
- Trích xuất từ một điều khoản quy chế ra JSON (đối tượng áp dụng, điều kiện, con số, ngoại lệ), validate bằng Pydantic; nếu lỗi thì gửi lại kèm thông báo lỗi để model tự sửa
- Viết 3 phiên bản prompt, chạy trên 20 điều khoản, so tỷ lệ JSON hợp lệ và đúng nội dung

## Tiêu chí đạt

- [ ] tỷ lệ JSON hợp lệ ≥ 98% sau cơ chế tự sửa; mỗi lần gọi có log chi phí.

## Lý thuyết cần đọc

- [ ] Cấu trúc request: system prompt, lượt hội thoại, giới hạn token đầu ra; streaming
- [ ] Structured output: JSON schema, tool/function calling để ép định dạng
- [ ] Context engineering: thứ tự thông tin, ví dụ few-shot, chỉ dẫn rõ ràng, phân tách dữ liệu và chỉ dẫn bằng thẻ
- [ ] Prompt là code: lưu trong file, có version, có test

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
