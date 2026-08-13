---
title: "Tổng quan Workshop & Kiến trúc Hệ thống AWS"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

### 1. Tổng quan dự án (Overview)

Mục tiêu của dự án **Startups Blogs** là xây dựng một nền tảng ứng dụng web doanh nghiệp tập trung, hỗ trợ quảng bá năng lực và kết nối cơ hội gọi vốn giữa các **Doanh nghiệp vừa và nhỏ (SMEs)**, **Chủ doanh nghiệp (Business Owners)**, **Startups** và các **Nhà đầu tư (Investors)**.

Hệ thống hoạt động theo mô hình **Khám phá & Kết nối Đầu tư (Investment Discovery & Connection)**:
- **Doanh nghiệp**: Đăng ký tài khoản, công bố **Hồ sơ doanh nghiệp (Business Profile)**, chỉ số tài chính và các tin gọi vốn (**`FundingOpportunity`**).
- **Nhà đầu tư**: Tra cứu, lọc doanh nghiệp theo ngành nghề/giai đoạn, theo dõi tin tức và gửi yêu cầu liên hệ trực tiếp (**`ContactRequest`**).
- **Quản trị viên (Admin & Moderator)**: Duyệt doanh nghiệp mới (**`PENDING`** → **`APPROVED`**) và duyệt các đề xuất thay đổi hồ sơ (**`ChangeProposal`**).

> 💡 **Lưu ý nghiệp vụ**: Nền tảng **không** thực thi các giao dịch chuyển tiền, thanh toán trực tuyến hay xử lý giao dịch cổ phần trên trang.

---

### 2. Mục tiêu bài học (Objectives)

- Nắm vững kiến trúc tổng quan của ứng dụng **Full-Stack** triển khai trên **AWS Cloud**.
- Hiểu rõ vai trò của từng tầng công nghệ: **React 19 Frontend**, **NestJS REST API**, **Prisma ORM**, **PostgreSQL** và **Amazon Cognito**.
- Định hình được phạm vi hạ tầng điện toán đám mây được tự động hóa bằng **Terraform**.

---

### 3. Kiến trúc hệ thống & Luồng dữ liệu (System Architecture)

Luồng dữ liệu chính của hệ thống được tổ chức theo các tầng xử lý:

```text
Trình duyệt Người dùng (React 19 SPA)
      ↓ HTTPS REST API
Amazon API Gateway (HTTP API)
      ↓ EC2 Security Group (Port 3000)
NestJS Backend (Amazon EC2 trong Public Subnet)
      ↓ Prisma ORM Client
Amazon RDS PostgreSQL (Private Subnet - Port 5432)
```

Dịch vụ đám mây hỗ trợ:
- **Amazon Cognito User Pool**: Quản lý đăng ký, đăng nhập và xác thực OTP Email.
- **Amazon S3** & **Amazon CloudFront**: Lưu trữ mã nguồn tĩnh Frontend và tài liệu/hình ảnh đính kèm (`POST /upload`).
- **Amazon CloudWatch** & **Amazon SNS**: Giám sát hiệu năng CPU của **Amazon EC2** và gửi cảnh báo qua Email.

---

### 4. Sơ đồ Kiến trúc Đám mây AWS

![Kiến trúc AWS của Startups Blogs](/images/workshop/aws-architecture.png)

*Hình 1. Kiến trúc triển khai tổng quan hệ thống Startups Blogs trên AWS Cloud.*

> **Lưu ý:** Sơ đồ mang tính minh họa tổng thể. Phiên bản Terraform hiện tại chưa triển khai Route 53 và RDS Multi-AZ.

---

### 5. Giao diện Người dùng Khám phá Doanh nghiệp

![Giao diện trang chủ](/images/workshop/frontend-homepage.png)

*Hình 2. Giao diện trang chủ của nền tảng Startups Blogs.*

---

### 6. Tóm tắt (Summary)

Trong phần này, chúng ta đã nắm bắt được mục tiêu nghiệp vụ của **Startups Blogs**, hiểu rõ ranh giới vai trò người dùng và hình dung được mô hình kết nối giữa **React 19**, **NestJS Backend**, **Amazon RDS PostgreSQL** và các dịch vụ đám mây **AWS**. Trong phần tiếp theo (5.2), chúng ta sẽ bắt đầu thiết lập môi trường phát triển cục bộ với **Docker Compose**.
