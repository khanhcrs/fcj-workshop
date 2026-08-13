---
title: "Thiết lập Môi trường Phát triển Cục bộ, Docker & PostgreSQL"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

### 1. Tổng quan (Overview)

Trước khi tiến hành cấu hình dịch vụ đám mây **Amazon Cognito** và triển khai hạ tầng AWS bằng **Terraform**, chúng ta cần chuẩn bị môi trường phát triển cục bộ (Local Development Environment). 

Việc đóng gói cơ sở dữ liệu **PostgreSQL** và bộ lưu trữ **MinIO** bằng **Docker Compose** giúp phát triển ứng dụng offline một cách nhất quán trước khi đẩy lên AWS Cloud.

---

### 2. Mục tiêu bài học (Objectives)

- Cài đặt và chuẩn bị các công cụ phát triển cần thiết (**Node.js**, **Docker**, **Terraform**, **AWS CLI**).
- Khởi chạy các container **PostgreSQL 15** và **MinIO** S3 local bằng **Docker Compose**.
- Kiểm tra trạng thái hoạt động của các cổng kết nối (**Port 5433** và **Port 9000/9001**).

---

### 3. Yêu cầu công cụ phần mềm (Prerequisites)

- **Node.js**: Phiên bản Node.js 18+ hoặc 20+ LTS.
- **Docker & Docker Desktop**: Công cụ đóng gói và quản lý container.
- **Terraform**: Phiên bản v1.5+ phục vụ tự động hóa hạ tầng đám mây.
- **AWS CLI**: Đã cài đặt và cấu hình default region **`us-east-1`** (N. Virginia).

---

### 4. Kiến trúc Container Cục bộ (Local Docker Architecture)

File `docker-compose.yml` trong dự án định nghĩa hai dịch vụ container chính:

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    container_name: startups_blogs_db
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgrespassword
      POSTGRES_DB: startups_blogs
    ports:
      - '5433:5432' # Mapped port 5433 trên Host tới 5432 trong Container

  minio:
    image: minio/minio:RELEASE.2023-09-04T19-57-37Z
    container_name: startups_blogs_minio
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadminpassword
    ports:
      - '9000:9000' # S3 API Endpoint
      - '9001:9001' # Web Console Interface
    command: server /data --console-address ":9001"
```

- **PostgreSQL 15 (`startups_blogs_db`)**: Cơ sở dữ liệu quan hệ cục bộ lắng nghe tại **Port 5433**.
- **MinIO (`startups_blogs_minio`)**: Bộ lưu trữ đối tượng tương thích với Amazon S3 API lắng nghe tại **Port 9000** (API) và **Port 9001** (Console).

---

### 5. Thực hành: Khởi chạy Hạ tầng Cục bộ

#### Bước 1: Mở Terminal và di chuyển vào thư mục dự án

```bash
cd /Users/khanhtran/Project/Startup_Blogs
```

#### Bước 2: Khởi chạy các dịch vụ Docker bằng Docker Compose

Chạy lệnh `docker compose up -d` để khởi tạo container ngầm.

```bash
docker compose up -d
```

#### Bước 3: Kiểm tra trạng thái container đang hoạt động

```bash
docker ps
```

---

### 6. Kết quả kỳ vọng (Expected Result)

Các container `startups_blogs_db` và `startups_blogs_minio` phải ở trạng thái **Running** (Healthy):

![Docker Compose](/images/workshop/docker-compose.png)

*Hình 3. Khởi chạy PostgreSQL và MinIO bằng Docker Compose.*

---

### 7. Xử lý sự cố thường gặp (Troubleshooting)

- **Lỗi đụng cổng 5433 hoặc 5432**: Kiểm tra xem máy cục bộ có dịch vụ PostgreSQL khác đang chạy hay không bằng lệnh `lsof -i :5433`.
- **Lỗi Docker daemon chưa khởi động**: Đảm bảo ứng dụng **Docker Desktop** đã được bật trước khi chạy `docker compose`.

---

### 8. Tóm tắt (Summary)

Chúng ta đã khởi chạy thành công môi trường **PostgreSQL** và **MinIO** cục bộ bằng **Docker Compose**. Ở bài thực hành tiếp theo (5.3), chúng ta sẽ tiến hành khởi tạo Schema cơ sở dữ liệu với **Prisma ORM** và nạp dữ liệu mẫu.