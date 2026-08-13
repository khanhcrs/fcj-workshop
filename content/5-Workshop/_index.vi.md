---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

#### Tổng quan (Overview)

Nội dung Workshop bao gồm **7 phần thực hành từng bước**, hướng dẫn chi tiết quy trình thiết kế, phát triển và tự động hóa hạ tầng ứng dụng web doanh nghiệp **Startups Blogs** trên nền tảng **AWS** (Region: **`us-east-1`**).

Hệ thống kết hợp bộ công nghệ hiện đại bao gồm **React 19 (TypeScript, Vite)** ở Frontend, **NestJS REST API** ở Backend, cơ sở dữ liệu quan hệ **Amazon RDS PostgreSQL**, lưu trữ đa phương tiện **Amazon S3**, mạng phân phối **Amazon CloudFront**, cửa ngõ API **Amazon API Gateway**, dịch vụ xác thực Cloud-Native **Amazon Cognito**, giám sát **Amazon CloudWatch**, và tự động hóa 100% bằng **Terraform (Infrastructure as Code)**.

#### Các điểm kỹ thuật trọng tâm (Key Highlights)

- **Tự động hóa Hạ tầng bằng Terraform**: Khai báo và khởi tạo toàn bộ tài nguyên AWS (**Amazon VPC**, **Amazon EC2**, **Amazon RDS**, **Amazon Cognito**, **Amazon API Gateway**, **Amazon S3**, **Amazon CloudFront**, **Amazon CloudWatch**, **Amazon SNS**) thông qua mã nguồn trong thư mục `terraform/`.
- **Ủy quyền Quản lý Danh tính Cloud-Native**: Mật khẩu người dùng không lưu trong **PostgreSQL** mà được ủy quyền hoàn toàn cho **Amazon Cognito User Pool**.
- **Cấu hình Public App Client**: Sử dụng **Cognito Public App Client** (**`generate_secret = false`**) phù hợp với mô hình ứng dụng web hiện đại.
- **Thẩm định Chữ ký số RSA bằng `aws-jwt-verify`**: NestJS Backend kiểm tra tính toàn vẹn và thẩm định chữ ký mã hóa **RS256** của **JWT** trực tiếp với tập khóa công khai **JWKS** từ Cognito **`us-east-1`**.
- **Phân quyền Hai tầng (Dual-Layer Authorization)**: Kết hợp giữa **`JwtAuthGuard`** (xác thực danh tính), **`RolesGuard`** (phân quyền vai trò **`ADMIN`**, **`MODERATOR`**, **`USER`**), và kiểm tra **Resource Ownership** (**`ownerId === userId`**) tại Service Layer.
- **Lưu trữ Đa phương tiện Amazon S3 (`POST /upload`)**: Tích hợp **`@aws-sdk/client-s3`** hỗ trợ tải ảnh logo, avatar, ảnh bìa doanh nghiệp lên **Amazon S3 Media Bucket** / **MinIO**.

#### Các phần thực hành của Workshop (Workshop Modules)

1. [5.1 Tổng quan Workshop & Kiến trúc Hệ thống AWS](5.1-Workshop-overview/)
2. [5.2 Thiết lập Môi trường Phát triển Cục bộ, Docker & PostgreSQL](5.2-Prerequiste/)
3. [5.3 Khởi tạo Schema Cơ sở Dữ liệu, Prisma ORM & Dữ liệu Mẫu](5.3-Cognito-setup/)
4. [5.4 Cấu hình Amazon Cognito User Pool, Public App Client & RBAC](5.4-Backend-integration/)
5. [5.5 Tích hợp NestJS REST API, Upload S3 & Thẩm định RSA JWT](5.5-Frontend-integration/)
6. [5.6 Tích hợp React 19 Frontend, Tìm kiếm Doanh nghiệp & Admin Dashboard](5.6-Security-review/)
7. [5.7 Tự động hóa Hạ tầng bằng Terraform, Giám sát & Dọn dẹp Tài nguyên](5.7-Cleanup/)