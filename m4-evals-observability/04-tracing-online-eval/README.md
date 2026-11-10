# 04-tracing-online-eval

> M4 · Evals và observability — Khối 4 · Tuần 30–32 · Observability và eval online

**Trạng thái:** ⬜ Chưa bắt đầu

## Mục tiêu

- Gắn tracing vào toàn bộ pipeline
- Dashboard: p50/p95 độ trễ theo bước, chi phí mỗi request, tỷ lệ phản hồi tiêu cực theo ngày
- Quy trình hằng tuần: lấy mẫu 20 trace thật, gán nhãn, câu sai đưa vào golden set
- Viết bài blog về toàn bộ quá trình eval

## Tiêu chí đạt

- [ ] từ một phản hồi “sai” của người dùng, mở được trace và chỉ ra bước gây lỗi trong dưới 5 phút.

## Lý thuyết cần đọc

- [ ] Tracing: span cho từng bước (viết lại truy vấn → truy xuất → rerank → sinh); OpenTelemetry
- [ ] Công cụ: Langfuse, Phoenix, Promptfoo, Inspect AI…; chọn một và làm sâu
- [ ] Cần ghi gì: version prompt, id chunk được truy xuất, token, độ trễ, chi phí, phản hồi người dùng
- [ ] Quyền riêng tư trong log: che thông tin cá nhân, thời gian lưu
- [ ] Metric online: tỷ lệ phản hồi tiêu cực, tỷ lệ từ chối, câu hỏi lặp lại; lấy mẫu log để gán nhãn

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
