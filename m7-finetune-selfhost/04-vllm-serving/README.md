# 04-vllm-serving

> M7 · Fine-tuning và self-hosting — Khối 4 · Tuần 58–60 · Self-hosting

**Trạng thái:** ⬜ Chưa bắt đầu

## Mục tiêu

- Thuê GPU cloud vài giờ, serve một model open-weight nhỏ bằng vLLM
- Load test bằng công cụ ở M6; đo token/giây, p95, chi phí mỗi 1.000 request
- Chạy eval M4 với model tự host thay cho API

## Tiêu chí đạt

- [ ] bản ghi quyết định “self-host hay API” với điểm hòa vốn theo số request mỗi tháng.

## Lý thuyết cần đọc

- [ ] Serve model bằng vLLM hoặc tương đương; continuous batching
- [ ] Lượng tử hóa (GPTQ, AWQ, GGUF): đánh đổi chất lượng và bộ nhớ
- [ ] Throughput và độ trễ; chi phí GPU theo giờ so với giá API theo token
- [ ] Lý do không vì chi phí: dữ liệu nhạy cảm, chạy offline, kiểm soát phiên bản

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
