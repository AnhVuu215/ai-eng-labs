# Hướng dẫn: Docker, CI/CD, deploy

> Thời lượng: 2 tuần · Đóng gói code của [03-redis-queue-reliability](../03-redis-queue-reliability/)

## Bạn sẽ làm ra gì

- `docker compose up` dựng toàn bộ: api, worker, postgres, redis
- GitHub Actions mỗi PR chạy: lint → mypy → test → build image
- Một URL công khai có HTTPS
- ADR đầu tiên

**Gợi ý tổ chức:** lab này nên là **repo riêng**, chính là repo dự án chính (RAG quy chế HUCE) sẽ lớn dần ở M3–M6. Lý do: CI và deploy gắn với một repo cụ thể. Trong thư mục lab này chỉ để README ghi link sang repo đó cùng kết quả đo.

---

## Bước 1: Dockerfile multi-stage (3 giờ)

```dockerfile
# ---- build stage: cài dependency ----
FROM python:3.12-slim AS builder
COPY --from=ghcr.io/astral-sh/uv:latest /uv /usr/local/bin/uv
WORKDIR /app
ENV UV_COMPILE_BYTECODE=1 UV_LINK_MODE=copy
COPY pyproject.toml uv.lock ./
RUN uv sync --frozen --no-dev --no-install-project   # layer cache: chỉ chạy lại khi lock đổi
COPY src ./src
RUN uv sync --frozen --no-dev

# ---- runtime stage: chỉ chứa thứ cần chạy ----
FROM python:3.12-slim
RUN useradd --create-home app
WORKDIR /app
COPY --from=builder --chown=app /app /app
USER app
ENV PATH="/app/.venv/bin:$PATH"
EXPOSE 8000
CMD ["uvicorn", "regulation_api.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

Lưu ý: thay `uv:latest` bằng một tag phiên bản cụ thể (ví dụ `uv:0.9.x`), vì build phải lặp lại được.

Thêm `.dockerignore`: `.venv`, `.git`, `__pycache__`, `tests`, `.env`.

**Đo:** `docker images` và ghi kích thước image. Thử build bằng image `python:3.12` (bản đầy đủ) một stage để so sánh.

✅ **Checkpoint:** `docker build -t regulation-api .` rồi `docker run` trả được `/docs`. **Commit.**

## Bước 2: Compose đầy đủ và healthcheck (2–3 giờ)

```yaml
services:
  api:
    build: .
    env_file: .env
    ports: ["8000:8000"]
    depends_on:
      db: { condition: service_healthy }
      redis: { condition: service_healthy }
      migrate: { condition: service_completed_successfully }
    healthcheck:
      test: ["CMD", "python", "-c", "import urllib.request; urllib.request.urlopen('http://localhost:8000/healthz')"]
      interval: 10s
  migrate:
    build: .
    command: alembic upgrade head
    depends_on: { db: { condition: service_healthy } }
  worker:
    build: .
    command: arq regulation_api.worker.WorkerSettings
    depends_on: [db, redis]
  db:
    image: postgres:17
    healthcheck: { test: ["CMD-SHELL", "pg_isready -U app"], interval: 5s }
  redis:
    image: redis:7
    healthcheck: { test: ["CMD", "redis-cli", "ping"], interval: 5s }
```

Endpoint health: `/healthz` (process còn sống) khác `/readyz` (kết nối được DB và Redis). Tự trả lời vì sao cần hai endpoint.

**Logging JSON:** dùng `structlog` hoặc `logging` với formatter JSON. Mỗi dòng log có `request_id`, `path`, `status`, `duration_ms`. Dùng middleware để sinh `request_id`.

✅ **Checkpoint:** máy sạch chạy `git clone` → `cp .env.example .env` → `docker compose up` là chạy được. **Commit.**

## Bước 3: GitHub Actions CI (3 giờ)

```yaml
# .github/workflows/ci.yml
name: ci
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:17
        env: { POSTGRES_USER: app, POSTGRES_PASSWORD: app, POSTGRES_DB: app_test }
        ports: ["5433:5432"]
        options: --health-cmd pg_isready --health-interval 5s
      redis:
        image: redis:7
        ports: ["6379:6379"]
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v6
      - run: uv sync --frozen
      - run: uv run ruff check . && uv run ruff format --check .
      - run: uv run mypy src
      - run: uv run pytest
  build:
    needs: test
    runs-on: ubuntu-latest
    permissions: { contents: read, packages: write }
    steps:
      - uses: actions/checkout@v4
      - uses: docker/login-action@v3
        with: { registry: ghcr.io, username: "${{ github.actor }}", password: "${{ secrets.GITHUB_TOKEN }}" }
      - uses: docker/build-push-action@v6
        with:
          push: ${{ github.ref == 'refs/heads/main' }}
          tags: ghcr.io/${{ github.repository }}:${{ github.sha }}
```

Kiểm tra lại phiên bản các action (`@v4`, `@v6`...) trên GitHub Marketplace trước khi dùng.

Bật **branch protection** cho `main`: bắt buộc CI xanh mới merge. Từ giờ làm việc qua PR, kể cả khi làm một mình. Đây là thói quen mà nhà tuyển dụng nhìn thấy được trong repo.

✅ **Checkpoint:** mở 1 PR thì thấy check xanh; cố tình làm hỏng 1 test thì thấy check đỏ. **Commit.**

## Bước 4: Deploy công khai có HTTPS (3–5 giờ)

Lựa chọn (tự so sánh giá và ghi vào ADR):

| Cách | Ưu | Nhược |
|---|---|---|
| VPS (Hetzner, DigitalOcean, Vultr, nhà cung cấp VN) + Docker Compose + Caddy | học được nhiều nhất, rẻ | tự lo bảo mật, cập nhật |
| PaaS (Fly.io, Railway, Render) | nhanh, có HTTPS sẵn | giới hạn free tier, ít kiểm soát |
| GitHub Student Developer Pack | có credit cloud miễn phí cho sinh viên | cần xác minh |

Với VPS, Caddy tự lấy chứng chỉ HTTPS:

```
# Caddyfile
api.tenmien.com {
    reverse_proxy api:8000
}
```

Checklist bảo mật VPS tối thiểu: tắt đăng nhập SSH bằng mật khẩu (chỉ dùng key) · bật firewall (`ufw allow 22,80,443`) · **không mở cổng 5432/6379 ra internet** · đặt `.env` trên server bằng tay, không commit.

Deploy tự động (tùy chọn): job `deploy` trong CI SSH vào server, chạy `docker compose pull && docker compose up -d`.

✅ **Checkpoint:** `curl https://<domain>/healthz` trả 200. **Commit.**

## Bước 5: ADR đầu tiên (1–2 giờ)

Tạo `docs/adr/0001-postgres-thay-vi-vector-db.md` trong repo dự án, theo [template](../../m8-senior-skills/adr/0000-template.md). Cần trả lời:
- Bối cảnh: dữ liệu bao nhiêu (vài nghìn khoản), team mấy người, ngân sách bao nhiêu.
- Phương án: Postgres + pgvector · Qdrant/Weaviate riêng · Elasticsearch.
- Quyết định và **khi nào sẽ xem lại** (ví dụ: > 1 triệu vector, hoặc p95 truy xuất > X ms).

## Bước 6: Kiểm chứng "người khác chạy được trong 10 phút"

Nhờ một bạn clone repo và làm theo README, **bấm giờ**, không giúp gì. Mỗi chỗ họ kẹt là một lỗi của README. Sửa xong mới tính là đạt.

## Kết quả đo cần ghi

| Chỉ số | Giá trị |
|---|---|
| Kích thước image: 1 stage `python:3.12` / multi-stage slim | |
| Thời gian build: lần đầu / khi chỉ đổi code (cache hit) | |
| Thời gian CI một lượt | |
| URL công khai | |
| Thời gian bạn bè chạy được từ README | |

## Lỗi hay gặp

| Triệu chứng | Nguyên nhân |
|---|---|
| api khởi động trước DB nên crash | thiếu `condition: service_healthy` |
| Đổi 1 dòng code mà cài lại toàn bộ dependency | `COPY . .` đặt trước bước cài dependency, làm mất layer cache |
| CI xanh trên máy, đỏ trên GitHub | test phụ thuộc thứ tự hoặc giờ địa phương, hoặc thiếu `uv.lock` |
| Caddy không lấy được chứng chỉ | DNS chưa trỏ về server, cổng 80 bị chặn |

## Tài liệu

- Docker docs: multi-stage builds, Compose `depends_on`/healthcheck
- uv docs: "Using uv in Docker"
- GitHub Actions docs: service containers, publishing Docker images
- Caddy docs: Automatic HTTPS
- DDIA chương 1–3
