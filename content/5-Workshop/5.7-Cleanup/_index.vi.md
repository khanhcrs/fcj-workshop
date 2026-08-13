---
title: "Tự động hóa Hạ tầng bằng Terraform, Giám sát & Dọn dẹp Tài nguyên"
date: 2024-01-01
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

### 1. Tổng quan (Overview)

Toàn bộ hạ tầng điện toán đám mây AWS của dự án **Startups Blogs** được khai báo và tự động hóa 100% bằng **Terraform (Infrastructure as Code)** trong thư mục `terraform/`.

Bài thực hành cuối cùng hướng dẫn quy trình khởi tạo hạ tầng AWS bằng Terraform, cấu hình giám sát với **Amazon CloudWatch**, và thực thi hủy tài nguyên (Resource Cleanup) để tránh phát sinh chi phí ngoài ý muốn.

---

### 2. Mục tiêu bài học (Objectives)

- Nắm vững quy trình khởi tạo hạ tầng đám mây AWS bằng mã nguồn **Terraform** IaC.
- Hiểu cách thiết lập **Amazon CloudWatch Alarm** giám sát chỉ số CPU Utilization của EC2 và gửi Email thông qua **Amazon SNS**.
- Thực thi quy trình dọn dẹp tài nguyên đám mây với `terraform destroy` và dọn dẹp môi trường cục bộ với `docker compose down -v`.

---

### 3. Cấu trúc Mã nguồn Tự động hóa Hạ tầng Terraform (`terraform/`)

Các file cấu hình Terraform trong thư mục `terraform/` quản lý 100% tài nguyên đám mây tại region **`us-east-1`**:

- **`vpc.tf`**: Tạo mạng ảo **Amazon VPC** (`10.0.0.0/16`), 2 Public Subnets, 2 Private Subnets, Internet Gateway và Security Groups (`ec2_sg` **Port 3000**, `rds_sg` **Port 5432**).
- **`ec2.tf`**: Khởi tạo máy chủ **Amazon EC2** Ubuntu cho NestJS Backend và gán **IAM Instance Profile** (`ec2_role`).
- **`rds.tf`**: Khởi tạo cơ sở dữ liệu quan hệ **Amazon RDS PostgreSQL** (`db.t3.micro`) trong Private Subnet biệt lập.
- **`cognito.tf`**: Khởi tạo **Amazon Cognito User Pool** và **Public App Client** (**`generate_secret = false`**).
- **`s3_cloudfront.tf`**: Khởi tạo **Amazon S3 Frontend Bucket**, **Amazon S3 Media Bucket** và **Amazon CloudFront** CDN Distribution.
- **`apigateway.tf`**: Khởi tạo **Amazon API Gateway** (HTTP API) định tuyến HTTPS proxy tới EC2.
- **`monitoring.tf`**: Khởi tạo **Amazon CloudWatch Metric Alarm** (`CPUUtilization >= 70%`), CloudWatch Dashboard và **Amazon SNS Topic** Email notifications (`alert_email`).

---

### 4. Thực hành: Khởi tạo Hạ tầng Đám mây AWS bằng Terraform

#### Bước 1: Mở Terminal và di chuyển vào thư mục terraform

```bash
cd /Users/khanhtran/Project/Startup_Blogs/terraform
```

#### Bước 2: Khởi tạo các provider Terraform

```bash
terraform init
```

#### Bước 3: Kiểm tra kế hoạch tạo tài nguyên

```bash
terraform plan
```

#### Bước 4: Thực thi tạo hạ tầng AWS

```bash
terraform apply
```

Gõ `yes` khi Terminal yêu cầu xác nhận để bắt đầu quá trình khởi tạo tài nguyên đám mây trên AWS (**`us-east-1`**).

---

### 5. Giám sát Hệ thống với Amazon CloudWatch (`monitoring.tf`)

File `monitoring.tf` triển khai giải pháp giám sát tự động:
- **CloudWatch Metric Alarm**: Tự động theo dõi chỉ số `CPUUtilization` của máy chủ EC2 Backend.
- **Amazon SNS Email Notification**: Gửi email thông báo tức thì tới địa chỉ `var.alert_email` khi CPU vượt ngưỡng 70%.
- **CloudWatch Dashboard**: Cung cấp biểu đồ trực quan giám sát hiệu năng **Amazon EC2** và **Amazon RDS PostgreSQL**.

---

### 6. Thực hành: Dọn dẹp Tài nguyên Đám mây & Môi trường Cục bộ

> ⚠️ **CẢNH BÁO NGUY HIỂM**: Lệnh **`terraform destroy`** sẽ vĩnh viễn xóa bỏ toàn bộ tài nguyên đám mây đã được khởi tạo trên AWS. Hãy chắc chắn rằng bạn đã hoàn thành kiểm thử và lưu trữ dữ liệu cần thiết trước khi thực thi.

#### Bước 1: Hủy toàn bộ hạ tầng AWS Đám mây bằng Terraform

```bash
cd /Users/khanhtran/Project/Startup_Blogs/terraform
terraform destroy
```

Gõ `yes` khi Terminal yêu cầu xác nhận để giải phóng hoàn toàn các tài nguyên trên AWS.

#### Bước 2: Dọn dẹp môi trường Container cục bộ

```bash
cd /Users/khanhtran/Project/Startup_Blogs
docker compose down -v
```

Lệnh `docker compose down -v` dừng các container `startups_blogs_db` và `startups_blogs_minio`, đồng thời xóa bỏ các Docker Volume tạm thời.

---

### 7. Tóm tắt & Tổng kết Workshop (Workshop Conclusion)

Chúc mừng bạn đã hoàn thành toàn bộ **7 phần thực hành** của Workshop! Qua khóa thực hành này, bạn đã:

1. Thấu hiểu kiến trúc ứng dụng web Enterprise phân tầng kết hợp **React 19**, **NestJS**, **PostgreSQL** và **Amazon Cognito**.
2. Tự động hóa 100% hạ tầng điện toán đám mây AWS (**Amazon VPC**, **Amazon EC2**, **Amazon RDS**, **Amazon Cognito**, **Amazon API Gateway**, **Amazon S3**, **Amazon CloudFront**, **Amazon CloudWatch**, **Amazon SNS**) bằng **Terraform (IaC)**.
3. Master quy trình cấu hình **Amazon Cognito Public App Client** và cơ chế đồng bộ nhóm **`ADMIN`**.
4. Triển khai các REST API phục vụ doanh nghiệp, tin gọi vốn, upload ảnh S3 và thẩm định chữ ký số RSA của JWT bằng **`aws-jwt-verify`**.
5. Rà soát bảo mật phân quyền hai tầng (**`JwtAuthGuard`**, **`RolesGuard`**, Resource Ownership) và nắm vững quy trình dọn dẹp tài nguyên an toàn.
