# 01-reliability-fault-injection

> M6 · Production, LLMOps và bảo mật — Khối 1 · Tuần 40–41 · Độ tin cậy

**Trạng thái:** ⬜ Chưa bắt đầu

## Mục tiêu

- Viết lớp giả lập lỗi: nhà cung cấp trả 500, timeout, trả rác
- Cài fallback sang model dự phòng và chế độ “chỉ truy xuất” khi mọi model lỗi
- Đặt SLO (ví dụ p95 dưới một ngưỡng bạn chọn, tỷ lệ lỗi dưới 1%) và đo

## Tiêu chí đạt

- [ ] tắt nhà cung cấp chính mà người dùng vẫn nhận được câu trả lời hoặc kết quả truy xuất; có số đo SLO.

## Lý thuyết cần đọc

- [ ] Timeout, retry, circuit breaker; fallback sang model hoặc nhà cung cấp khác
- [ ] Suy giảm nhẹ nhàng: khi LLM chết, vẫn trả về các điều khoản liên quan nhất thay vì trang lỗi
- [ ] Ngân sách độ trễ cho từng bước; streaming để giảm thời gian chờ cảm nhận
- [ ] SLO: định nghĩa “hệ thống ổn” bằng con số

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
