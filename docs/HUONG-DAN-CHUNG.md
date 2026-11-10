# Hướng dẫn chung: cài đặt và quy trình làm lab

Đọc file này một lần trước khi bắt đầu M1. Mỗi lab có `GUIDE.md` riêng; file này gom những thứ lặp lại ở mọi lab.

## 1. Cài công cụ một lần

| Công cụ | Dùng để | Cài (Windows) |
|---|---|---|
| Python 3.12+ | ngôn ngữ chính | `winget install Python.Python.3.12` |
| uv | quản lý dự án, môi trường ảo, dependency | `winget install astral-sh.uv` |
| Git + GitHub CLI | lưu và đẩy code | đã có |
| Docker Desktop | Postgres, Redis, deploy (từ M1 khối 2) | `winget install Docker.DockerDesktop` |
| VS Code | editor | extension: Python, Ruff, Mypy Type Checker |

Kiểm tra:

```bash
python --version   # 3.12.x
uv --version
docker --version
```

## 2. Mỗi lab là một dự án uv độc lập

```bash
cd m1-software-foundations/01-doc-loader
uv init --package --name doc_loader .     # tạo pyproject.toml + src/doc_loader/
uv add pydantic                            # dependency chạy
uv add --dev pytest pytest-cov mypy ruff   # dependency dev
uv run pytest                              # luôn chạy qua `uv run`
```

Vì sao tách từng lab: lab sau có thể dùng phiên bản thư viện khác lab trước mà không vỡ. Khi lab sau cần code của lab trước (ví dụ `regulation-api` dùng `doc-loader`), thêm dependency dạng đường dẫn:

```bash
uv add --editable ../01-doc-loader
```

### Cấu hình chất lượng code dùng chung

Dán vào `pyproject.toml` của mỗi lab:

```toml
[tool.ruff]
line-length = 100
[tool.ruff.lint]
select = ["E", "F", "I", "B", "UP", "SIM", "ASYNC"]

[tool.mypy]
strict = true

[tool.pytest.ini_options]
addopts = "-q --cov --cov-report=term-missing"
```

Lệnh kiểm tra trước mỗi commit:

```bash
uv run ruff check . --fix && uv run ruff format . && uv run mypy src && uv run pytest
```

## 3. Bí mật và API key

- Key nằm trong file `.env` ở thư mục lab. File này **đã bị `.gitignore` chặn**, không bao giờ commit.
- Commit một file `.env.example` chỉ có tên biến, không có giá trị:

```bash
ANTHROPIC_API_KEY=
DATABASE_URL=postgresql+psycopg://app:app@localhost:5432/app
```

- Đọc biến môi trường bằng `pydantic-settings`, không hard-code.
- Lỡ commit key: **thu hồi key ngay** trên trang nhà cung cấp, sau đó mới xóa khỏi git. Xóa commit thôi thì không đủ, vì key đã lộ.

## 4. Vòng lặp làm một khối 2 tuần

```
Ngày 1–3   Lý thuyết → viết notes/m<N>-<chu-de>.md bằng lời của mình (1 trang)
Ngày 4–11  Lab theo GUIDE.md, commit sau mỗi bước có checkpoint ✅
Ngày 12–13 Đo, điền bảng "Kết quả đo" trong README của lab
Ngày 14    Tự kiểm tra (self-check.md), cập nhật trạng thái ⬜→✅ ở README gốc
```

Kẹt quá 2 giờ ở một chỗ: ghi lại đã thử gì vào phần "Ghi chép" của README, rồi hỏi (AI, bạn bè, issue của thư viện). Ghi chép này về sau thành tư liệu viết blog.

## 5. Quy ước commit

Viết theo dạng `<module>/<lab>: <làm gì>`, dùng động từ, đủ cụ thể để đọc lại sau 6 tháng vẫn hiểu:

```
m1/doc-loader: tách chương-điều-khoản bằng regex, 12 test
m1/doc-loader: thêm retry có jitter cho tải URL
m3/rag-v1: đo recall@5 = 0.62 trên 50 câu
```

Commit nhỏ và thường xuyên, ít nhất mỗi lần đạt một checkpoint ✅. Push cuối mỗi buổi học.

## 6. Ghi "Kết quả đo" thế nào cho có giá trị

Nhà tuyển dụng đọc phần này đầu tiên. Mỗi số đo phải có:

- **Điều kiện đo**: máy gì, dữ liệu bao nhiêu, chạy mấy lần.
- **So với cái gì**: baseline, phiên bản trước.
- **Độ bất định** nếu có: khoảng tin cậy, độ lệch giữa các lần chạy.

Ví dụ tốt: `p95 tìm kiếm: 4.1 ms (có GIN index) vs 182 ms (không index), 10k điều khoản, 200 truy vấn, laptop i5`.
Ví dụ kém: `tìm kiếm nhanh hơn nhiều`.

## 7. Dùng LLM API (từ M2 khối 5 trở đi)

- Các ví dụ dùng Anthropic Python SDK (`uv add anthropic`). Mặc định model là `claude-opus-5-5`. Đặt model qua biến môi trường `LLM_MODEL` để đổi mà không sửa code. Khi chạy hàng trăm lần (eval, judge, sinh dữ liệu) bạn có thể tự chọn model rẻ hơn như `claude-haiku-5-5` ($0.10/$0.50 mỗi 1M token vào/ra, so với $4/$20 của Opus 5.5). Đây là quyết định của bạn, nên ghi lại vào ADR.
- Lộ trình yêu cầu ít nhất 2 nhà cung cấp. Nhà cung cấp thứ hai bạn tự chọn và làm theo tài liệu chính thức của họ.
- **Đặt giới hạn chi tiêu** (spend limit) trên trang quản lý của nhà cung cấp trước khi chạy vòng lặp lớn.
- Model Claude đời mới **không nhận `temperature`/`top_p`/`top_k`** (trả lỗi 400). Lab nào cần sampling (M2 khối 5) phải dùng model open-weight chạy local; GUIDE của lab đó có hướng dẫn.
