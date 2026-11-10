# 03-eval-in-ci

> M4 · Evals và observability — Khối 3 · Tuần 28–29 · Eval trong CI và vòng lặp thử nghiệm

**Trạng thái:** ⬜ Chưa bắt đầu

## Mục tiêu

- GitHub Action chạy bộ eval nhanh (30–40 câu) khi PR đụng tới prompt, retrieval hoặc model; bình luận kết quả lên PR
- Bộ đầy đủ chạy hằng đêm, lưu lịch sử
- Thực hiện ít nhất 5 thử nghiệm cải thiện (chunking, prompt, rerank, model, viết lại truy vấn); mỗi thử nghiệm ghi giả thuyết → thay đổi → kết quả có khoảng tin cậy

## Tiêu chí đạt

- [ ] biểu đồ chất lượng qua ≥ 5 phiên bản; ít nhất một thử nghiệm thất bại được ghi lại trung thực (đây là bằng chứng bạn đo thật).

## Lý thuyết cần đọc

- [ ] Regression test cho hệ thống xác suất: ngưỡng, độ dao động giữa các lần chạy
- [ ] Bộ eval nhanh (chạy mỗi PR) và bộ đầy đủ (chạy hằng đêm); chi phí chạy eval
- [ ] So sánh hai phiên bản trên cùng câu hỏi bằng paired bootstrap
- [ ] Thiết kế thử nghiệm: đổi một biến mỗi lần

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
