# 04-docker-ci-deploy

> M1 · Nền móng phần mềm — Khối 4 · Tuần 7–8 · Docker, CI/CD, deploy

**Trạng thái:** ⬜ Chưa bắt đầu

## Mục tiêu

- Compose gồm api, worker, postgres, redis; một lệnh dựng toàn bộ
- CI: lint, kiểm tra kiểu, test, build image mỗi PR
- Deploy lên một VPS hoặc dịch vụ cloud, có domain và HTTPS
- Viết ADR đầu tiên: vì sao chọn Postgres thay vì vector DB riêng ở giai đoạn này

## Tiêu chí đạt

- [ ] URL công khai chạy được; CI xanh; người khác clone repo và chạy được trong 10 phút theo README.

## Lý thuyết cần đọc

- [ ] Theo roadmap Docker trước: multi-stage build, Compose nhiều service, healthcheck
- [ ] GitHub Actions: test, build image, đẩy lên registry
- [ ] Logging có cấu trúc (JSON), health check, biến môi trường và secret
- [ ] System design cơ bản: load balancer, cache, queue, nhất quán dữ liệu

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
