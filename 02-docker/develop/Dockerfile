# ============================================================
# BASIC Dockerfile — Single-stage, dễ đọc, dễ hiểu
# Dùng làm điểm khởi đầu trước khi học multi-stage
# ============================================================

# Bước 1: Chọn base image
# Alpine base để tránh lỗi tương thích image trên môi trường hiện tại
FROM alpine:3.19

RUN apk add --no-cache python3 py3-pip

# Bước 2: Đặt working directory trong container
WORKDIR /app

# Bước 3: Copy requirements TRƯỚC (Docker layer cache)
COPY 02-docker/develop/requirements.txt .

# Bước 4: Cài dependencies
RUN pip3 install --break-system-packages --no-cache-dir --user -r requirements.txt

# Bước 5: Copy code vào container
COPY 02-docker/develop/app.py .

# Bước 6: Copy mock utils (from build context root)
RUN mkdir -p utils
COPY utils/mock_llm.py utils/

# Bước 7: Expose port để documentation (không bắt buộc)
EXPOSE 8000

# Bước 8: Command mặc định khi container start
CMD ["python3", "app.py"]

# ============================================================
# Để build và chạy:
#   cd ../../  # Go to project root
#   docker build -f 02-docker/develop/Dockerfile -t agent-develop .
#   docker run -p 8000:8000 agent-develop
#   curl http://localhost:8000/health
# ============================================================
