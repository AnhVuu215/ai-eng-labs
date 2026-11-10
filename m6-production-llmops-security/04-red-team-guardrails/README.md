# 04-red-team-guardrails

> M6 · Production, LLMOps và bảo mật — Khối 4 · Tuần 46–48 · Bảo mật và guardrails

**Trạng thái:** ⬜ Chưa bắt đầu

## Mục tiêu

- Viết bộ 30 tấn công: injection trực tiếp, injection trong tài liệu được truy xuất, cố moi system prompt, cố đọc dữ liệu sinh viên khác qua agent
- Chạy, ghi tỷ lệ tấn công thành công, vá, chạy lại
- Đưa bộ tấn công vào CI như một bộ eval
- Viết báo cáo bảo mật công khai (không lộ cách khai thác chưa vá)

## Tiêu chí đạt

- [ ] có báo cáo trước/sau; bộ tấn công chạy tự động mỗi lần thay đổi prompt hoặc tool.

## Lý thuyết cần đọc

- [ ] OWASP Top 10 cho ứng dụng LLM: prompt injection trực tiếp và gián tiếp, lộ thông tin nhạy cảm, xử lý output không an toàn, agent làm quá quyền
- [ ] Phân quyền tài liệu trong RAG: lọc theo quyền trước khi đưa vào context
- [ ] Che thông tin cá nhân trong log và trong context
- [ ] Output của model là dữ liệu không tin cậy: không render thẳng thành HTML, không chạy thẳng thành lệnh

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
