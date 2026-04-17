# Day 12 Lab - Câu Trả Lời Mission

## Part 1: Localhost vs Production

### Exercise 1.1: Các anti-pattern tìm thấy trong develop/app.py
1. Hardcode secret (`OPENAI_API_KEY`) trực tiếp trong source code.
2. Hardcode chuỗi kết nối database kèm credential trong source code.
3. Cố định port (`8000`) và chỉ bind `host="localhost"`, khó deploy lên cloud/container.
4. Bật `reload=True` cố định, không an toàn cho môi trường production.
5. Thiếu endpoint `/health` nên platform không thể kiểm tra liveness.
6. Dùng `print` để log và còn log cả thông tin nhạy cảm.
7. Không có xử lý graceful shutdown cho SIGTERM.
8. Config để rải rác thành hằng số trong code, không quản lý tập trung theo môi trường.

### Exercise 1.2: Chạy bản basic
- Đã cài dependencies từ `01-localhost-vs-production/develop/requirements.txt`.
- Đã khởi chạy ứng dụng bằng `python app.py`.
- Server chạy thành công tại `http://localhost:8000`.
- Ghi nhận về hành vi endpoint:
  - Bản develop hiện tại xử lý dữ liệu khác với pattern JSON body của bản production.
  - Ứng dụng chạy được nhưng chưa đạt chuẩn production-ready.

### Exercise 1.3: Bảng so sánh

| Tiêu chí | Develop (basic) | Production (advanced) | Vì sao quan trọng? |
|---------|------------------|------------------------|--------------------|
| Quản lý config | Hardcode hằng số trong `app.py` | Dùng `Settings` tập trung trong `production/config.py`, đọc từ env vars (`HOST`, `PORT`, `DEBUG`, ...) | Giữ được tính nhất quán giữa dev/staging/prod, không cần sửa code khi đổi môi trường. |
| Quản lý secrets | Secret nằm trong source (`OPENAI_API_KEY`, DB URL) và còn bị log | Secret đọc từ env vars, có validate qua `Settings.validate()` | Tránh lộ credential, hỗ trợ xoay vòng secret an toàn. |
| Contract request | `POST /ask` kiểu function arg (dễ gây lệch phía client) | `POST /ask` parse JSON body, validate rõ ràng, thiếu trường trả 422 | Tăng độ tin cậy API và giúp debug lỗi dễ hơn. |
| Logging | `print()` debug, có log secret | Structured JSON logging bằng `logging` + `json.dumps`, không log secret | Log dễ truy vấn/giám sát và an toàn hơn trong production. |
| Health check | Không có liveness/readiness endpoint | Có `GET /health` và `GET /ready` | Bắt buộc cho cloud platform/load balancer để auto-healing và route traffic đúng. |
| Shutdown | Không xử lý SIGTERM | Có SIGTERM handler + lifecycle startup/shutdown | Giảm rớt request đang xử lý khi restart/deploy. |
| Network binding | Chỉ `host="localhost"` | `host=settings.host` mặc định `0.0.0.0` | Bắt buộc để nhận traffic từ container và cloud ingress. |
| Quản lý port | Cố định `port=8000` | `port=settings.port` đọc từ env | Phù hợp platform cloud inject `PORT` động (Railway/Render/Cloud Run). |
| CORS | Chưa cấu hình | Có CORS middleware, cấu hình qua `ALLOWED_ORIGINS` | Kiểm soát truy cập từ trình duyệt, nâng mức an toàn API. |
| Quản lý vòng đời app | Không có startup/shutdown lifecycle | Có `lifespan` để quản lý readiness và cleanup | Tăng độ ổn định vận hành và khả năng quan sát hệ thống. |

### Kết luận Part 1
Bản develop thể hiện các anti-pattern phổ biến kiểu "chạy trên máy em thì ổn". Bản production áp dụng tốt các thực hành 12-factor và cloud-native: tách config ra môi trường, logging an toàn, có health/readiness probes, graceful shutdown, và host/port phù hợp triển khai thực tế.

## Part 2: Docker Containerization

### Exercise 2.1: Trả lời câu hỏi về Dockerfile basic
1. Base image là gì?
- Theo bản mẫu ban đầu: `python:3.11`.
- Theo bản chạy thực tế trong môi trường hiện tại: đã chuyển sang `alpine:3.19` và cài `python3`, `py3-pip` để tránh lỗi runtime `exec format error`.

2. Working directory là gì?
- Working directory là `/app` (khai báo bằng `WORKDIR /app`).

3. Tại sao `COPY requirements.txt` trước rồi mới copy source code?
- Để tận dụng Docker layer cache: nếu code thay đổi nhưng dependency không đổi thì layer cài pip không phải chạy lại, build nhanh hơn.

4. CMD và ENTRYPOINT khác nhau thế nào?
- `CMD` là lệnh mặc định có thể override khi chạy container.
- `ENTRYPOINT` thường dùng để cố định executable chính, khó bị thay thế hơn.
- Bản chạy hiện tại dùng `CMD ["python3", "app.py"]`.

### Exercise 2.2: Build và run bản develop
- Lệnh chuẩn (chạy từ root project):
  - `docker build -f 02-docker/develop/Dockerfile -t my-agent:develop .`
  - `docker run -p 8000:8000 my-agent:develop`
  - Test: `curl http://localhost:8000/health`

- Kết quả chạy thực tế:
  - Build develop: thành công.
  - Chạy container develop: thành công (`agent-dev`).
  - Test endpoint:
    - `GET /health` trả về JSON `{"status":"ok","uptime_seconds":...,"container":true}`.
    - `POST /ask?question=Hello Docker` trả về JSON có trường `answer`.
  - Kích thước image develop (inspect): `40740043` bytes (xấp xỉ 38.85 MB).

### Exercise 2.3: Multi-stage build (production)
- File tham chiếu: [02-docker/production/Dockerfile](02-docker/production/Dockerfile).

- Stage 1 (`AS builder`):
  - Bản chạy hiện tại dùng `alpine:3.19`, cài Python/pip và dependencies vào `/root/.local`.

- Stage 2 (`AS runtime`):
  - Dùng lại `alpine:3.19` nhẹ.
  - Copy package đã cài từ builder, copy code cần chạy.
  - Tạo non-root user `appuser`, chạy container không dùng root.
  - Có `HEALTHCHECK`.

- Vì sao image nhỏ hơn?
  - Runtime không mang theo build tools/layer tạm của giai đoạn build.
  - Chỉ giữ thành phần cần để chạy app.

- Lệnh so sánh size:
  - `docker build -f 02-docker/production/Dockerfile -t my-agent:advanced .`
  - `docker images | findstr my-agent`
  - Kết quả chạy thực tế:
    - Build advanced: thành công.
    - Kích thước image advanced (inspect): `31515028` bytes (xấp xỉ 30.06 MB).
    - Advanced nhỏ hơn develop khoảng 8.79 MB (xấp xỉ giảm 22.6%).

### Exercise 2.4: Docker Compose stack
- File tham chiếu: [02-docker/production/docker-compose.yml](02-docker/production/docker-compose.yml).

- Các service trong stack:
  1. `agent`: FastAPI app
  2. `redis`: cache/session/rate limit
  3. `qdrant`: vector database
  4. `nginx`: reverse proxy/load balancer

- Luồng tổng quan:
  - Client -> Nginx -> Agent
  - Agent -> Redis và Qdrant

- Lệnh chạy/test:
  - `docker compose -f 02-docker/production/docker-compose.yml up`
  - `curl http://localhost/health`
  - `curl http://localhost/ask -X POST -H "Content-Type: application/json" -d "{\"question\":\"Explain microservices\"}"`
  - `docker compose -f 02-docker/production/docker-compose.yml down`

- Trạng thái chạy thực tế:
  - Compose đã tạo được network và volumes.
  - Bị chặn khi pull `redis:7-alpine` do Docker Hub unauthenticated pull rate limit.
  - Vì vậy chưa hoàn tất full stack `agent + redis + qdrant + nginx` trong phiên này.

### Ghi chú Part 2
- Đã hoàn tất build/run/test thực tế cho image develop và advanced.
- Đã có số liệu kích thước image từ `docker image inspect`.
- Riêng phần compose full stack bị chặn bởi Docker Hub rate limit khi pull image public.

## Part 3: Cloud Deployment

### Exercise 3.1: Render deployment (lựa chọn chính)
- Trạng thái chuẩn bị:
  - Đã chạy local thành công trong thư mục `03-cloud-deployment/railway` để xác nhận app hoạt động.
  - `GET /health` trả về trạng thái `ok`.
  - `POST /ask` hoạt động khi gửi JSON body đúng format (`{"question": "..."}`).

- Triển khai Render:
  1. Push code lên GitHub.
  2. Vào Render Dashboard, chọn New -> Blueprint.
  3. Chọn repository và để Render đọc file `03-cloud-deployment/render/render.yaml`.
  4. Thiết lập environment variables cần thiết trên Render.
  5. Deploy và lấy public URL.

### Exercise 3.2: Railway deployment (tham khảo)
- Có sẵn cấu hình `railway.toml` trong project.
- Có thể dùng Railway nếu cần phương án thay thế Render.

### Exercise 3.3: Cloud Run (optional)
- Đã có template `cloudbuild.yaml` và `service.yaml` trong thư mục `production-cloud-run`.
- Bài này optional, có thể làm sau khi hoàn tất Render.

### Kết luận Part 3
- Kỹ thuật local của P3 đã pass (health + ask).
- Hướng triển khai chính: Render qua dashboard (không phụ thuộc CLI).
- Bước còn lại là lấy URL public sau deploy và điền vào `DEPLOYMENT.md`.
