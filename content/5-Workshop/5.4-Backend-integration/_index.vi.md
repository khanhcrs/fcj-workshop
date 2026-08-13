---
title: "Cấu hình Amazon Cognito User Pool, Public App Client & RBAC"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

### 1. Tổng quan (Overview)

**Amazon Cognito User Pool** cung cấp giải pháp quản lý danh tính và xác thực Cloud-Native cho ứng dụng **Startups Blogs** tại region **`us-east-1`**.

Bài thực hành này hướng dẫn thiết lập **Cognito User Pool**, tạo **Public App Client** (**`generate_secret = false`**), cấu hình luồng xác thực **`USER_PASSWORD_AUTH`**, và khởi tạo Cognito User Group **`ADMIN`** để đồng bộ phân quyền hệ thống.

> ⚠️ **Lưu ý kiến trúc quan trọng**: Ứng dụng web hiện tại sử dụng **Public App Client** (không tạo Client Secret Key) để tương thích chuẩn xác với mô hình SPA. Do đó, hệ thống **không** sử dụng `SecretHash` hay `COGNITO_CLIENT_SECRET`.

---

### 2. Mục tiêu bài học (Objectives)

- Khởi tạo **Amazon Cognito User Pool** hỗ trợ đăng nhập bằng Email và xác thực OTP 6 chữ số.
- Tạo App Client cấu hình dưới dạng **Public App Client** (**`generate_secret = false`**).
- Khởi tạo Cognito User Group **`ADMIN`** phục vụ phân quyền tài khoản Quản trị viên.
- Thẩm định địa chỉ **JWKS** endpoint (`.well-known/jwks.json`) tại region **`us-east-1`**.

---

### 3. Thực hành: Cấu hình Amazon Cognito trên AWS Console / Terraform

#### Bước 1: Khởi tạo Cognito User Pool
1. Đăng nhập **AWS Management Console** và chọn region **`us-east-1`** (N. Virginia) (hoặc xem cấu hình tự động tại `terraform/cognito.tf`).
2. Mở dịch vụ **Amazon Cognito** -> Click **Create user pool**.
3. Under **Authentication providers**: Chọn **Cognito user pool**.
4. Under **Sign-in options**: Chọn **Email**.
5. Under **Password policy**: Chọn độ dài tối thiểu 8 ký tự, hỗ trợ chữ hoa, chữ thường và chữ số.

![Tổng quan Cognito User Pool](/images/workshop/cognito-user-pool-overview.png)

*Hình 7. Tổng quan Amazon Cognito User Pool tại region us-east-1.*

#### Bước 2: Tạo Public App Client (No Client Secret)
1. Trong mục **App clients**: Đặt tên client `startups-blogs-app`.
2. **Client type**: Chọn **Public client**.
3. **Client secret**: Đảm bảo chọn **Don't generate a client secret** (**`generate_secret = false`**).
4. **Authentication flows**: Bật **`ALLOW_USER_PASSWORD_AUTH`** và **`ALLOW_REFRESH_TOKEN_AUTH`**.

![Cấu hình Cognito Public App Client](/images/workshop/cognito-app-client.png)

*Hình 8. Cấu hình Cognito Public App Client của Startups Blogs.*

#### Bước 3: Tạo Cognito User Group ADMIN
1. Vào tab **User management** -> Chọn **Groups** -> Click **Create group**.
2. Đặt tên Group: **`ADMIN`**.
3. Người dùng thuộc nhóm **`ADMIN`** sẽ được mã hóa claim `cognito:groups: ["ADMIN"]` trong **JWT** token.

![Cấu hình nhóm ADMIN](/images/workshop/cognito-admin-group.png)

*Hình 9. Cấu hình nhóm ADMIN trong Amazon Cognito User Pool.*

#### Bước 4: Kiểm tra Danh sách Người dùng

<!-- ================================================== -->
<!-- IMAGE PLACEHOLDER: COGNITO_USERS_SANITIZED -->
<!-- ================================================== -->

> **Hình 10. Danh sách người dùng đã xác thực trong Cognito User Pool.**  
> *Ảnh sẽ được bổ sung sau khi ẩn thông tin email cá nhân.*

<!-- ================================================== -->

---

### 4. Thông số Cấu hình Môi trường

Ghi lại các thông số sau để điền vào file cấu hình môi trường `.env` của backend NestJS:

```env
AWS_REGION=us-east-1
COGNITO_USER_POOL_ID=us-east-1_xxxxxxxxx
COGNITO_CLIENT_ID=xxxxxxxxxxxxxxxxxxxxxxxxxx
```

---

### 5. Tóm tắt (Summary)

Chúng ta đã khởi tạo xong **Amazon Cognito User Pool** ở chế độ **Public App Client** và tạo nhóm **`ADMIN`**. Ở phần tiếp theo (5.5), chúng ta sẽ tích hợp dịch vụ xác thực này vào **NestJS REST API** bằng thư viện **`aws-jwt-verify`**.
