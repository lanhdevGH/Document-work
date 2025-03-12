# Quy ước viết file Docker và Docker Compose

## I. Quy ước cho Dockerfile

### 1. Cấu trúc và Tổ chức
- Bắt đầu với FROM làm lệnh đầu tiên
- Nhóm các lệnh theo chức năng (cài đặt phụ thuộc, cấu hình, sao chép mã nguồn, v.v.)
- Đặt các lệnh ít thay đổi ở đầu tệp để tận dụng bộ nhớ cache Docker
- Sử dụng dấu backslash (\\) để chia các lệnh dài thành nhiều dòng

### 2. Lệnh và Cú pháp
- Viết hoa các từ khóa Docker (FROM, RUN, COPY, CMD, v.v.)
- Sử dụng LABEL để thêm metadata (maintainer, version, description)
- Ưu tiên COPY hơn ADD trừ khi cần giải nén tự động
- Kết hợp các lệnh RUN bằng `&&` để giảm số lớp image
- Chỉ định phiên bản cụ thể trong tag image (ví dụ: `FROM node:18-alpine` thay vì `FROM node:latest`)

### 3. Bảo mật và Hiệu suất
- Sử dụng USER để chạy container với quyền không phải root
- Sử dụng .dockerignore để loại trừ các tệp không cần thiết
- Dọn dẹp cache sau khi cài đặt gói (ví dụ: apt-get clean, pip cache purge, npm cache clean)
- Giảm kích thước image bằng cách sử dụng các base image nhỏ như Alpine

## II. Quy ước cho Docker Compose

### 1. Cấu trúc Tệp
- Đặt phiên bản ở đầu tệp (version: '3.8')
- Nhóm các dịch vụ theo chức năng hoặc theo sự phụ thuộc
- Đặt services, volumes, networks theo thứ tự đó
- Sử dụng dấu gạch ngang (-) cho các mục trong danh sách và thụt lề 2 dấu cách

### 2. Đặt tên và Quy ước
- Sử dụng tên dịch vụ mô tả (web, api, db) thay vì tên container
- Đặt tên rõ ràng cho volumes và networks
- Nhất quán với kiểu đặt tên (snake_case hoặc kebab-case)
- Sắp xếp các tùy chọn dịch vụ theo thứ tự logic (image, build, volumes, ports, environment)

### 3. Biến Môi trường và Cấu hình
- Sử dụng tệp .env cho các biến môi trường
- Ưu tiên tệp môi trường (env_file) hơn định nghĩa trực tiếp (environment)
- Đặt tên biến môi trường theo SERVICE_VARIABLE_NAME
- Chỉ định giá trị mặc định khi cần thiết

### 4. Quản lý Phụ thuộc
- Sử dụng depends_on để xác định thứ tự khởi động
- Sử dụng healthcheck để đảm bảo dịch vụ đã sẵn sàng
- Xác định restart policy phù hợp (no, always, on-failure, unless-stopped)

## III. Ví dụ về Convention Đầy đủ

### Dockerfile
```dockerfile
FROM node:18-alpine

LABEL maintainer="your.email@example.com"
LABEL version="1.0"
LABEL description="Application description"

# Cài đặt phụ thuộc
RUN apk add --no-cache \
    python3 \
    make \
    g++ \
    && rm -rf /var/cache/apk/*

# Cấu hình ứng dụng
WORKDIR /app

# Cài đặt phụ thuộc ứng dụng
COPY package*.json ./
RUN npm ci && npm cache clean --force

# Sao chép mã nguồn
COPY . .

# Xây dựng ứng dụng
RUN npm run build

# Cấu hình người dùng và cổng
EXPOSE 3000
USER node

# Lệnh khởi động
CMD ["npm", "start"]
```

### Docker Compose file
```yaml
# docker-compose.yml
version: '3.8'

services:
  api:
    build:
      context: ./api
      dockerfile: Dockerfile
    image: myapp/api:1.0
    volumes:
      - ./api:/app
      - api_node_modules:/app/node_modules
    ports:
      - "3000:3000"
    env_file:
      - .env
    environment:
      - NODE_ENV=development
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped

  db:
    image: postgres:14-alpine
    volumes:
      - db_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=${DB_USER}
      - POSTGRES_PASSWORD=${DB_PASSWORD}
      - POSTGRES_DB=${DB_NAME}
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
      interval: 5s
      timeout: 5s
      retries: 5

volumes:
  api_node_modules:
  db_data:

networks:
  default:
    name: myapp_network
```

## IV. Nguyên tắc thụt dòng trong Docker Compose file

1. Docker Compose sử dụng định dạng YAML, vì vậy việc thụt dòng rất quan trọng để xác định cấu trúc.

2. Mỗi cấp độ lồng nhau thường được thụt vào 2 dấu cách.

3. Các quy tắc thụt dòng cơ bản:
   - Các khóa cấp cao nhất (version, services, networks, volumes) không được thụt vào
   - Các khóa con được thụt vào 2 dấu cách so với khóa cha
   - Danh sách (mảng) các giá trị thường được thụt vào và mỗi phần tử bắt đầu bằng dấu gạch ngang (-)

Lưu ý: Lỗi thường gặp trong thụt dòng có thể dẫn đến lỗi cú pháp khi Docker Compose cố gắng phân tích tệp. Nếu bạn đang gặp vấn đề với tệp Docker Compose, hãy kiểm tra cẩn thận việc thụt dòng của bạn.
