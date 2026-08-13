---
title: "Các bài blogs đã đăng"
date: 2026-08-13
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

Trong quá trình nghiên cứu và thực tập tại chương trình **First Cloud AI Journey (FCAJ)**, em đã biên soạn 3 bài viết kỹ thuật chuyên sâu xoay quanh chủ đề **Kiến trúc Xác thực Đám mây AWS Amazon Cognito**, **Bảo mật JWT Token với aws-jwt-verify (`us-east-1`)**, và **Quản lý Session qua HttpOnly Cookie & Phân quyền RBAC cho ứng dụng Startups Blogs**.

---

### [Blog 1: Tại sao không nên gọi trực tiếp API xác thực từ Frontend? – Bài toán với Amazon Cognito, React và NestJS](3.1-Blog1/)

Bài viết phân tích lý do kiến trúc không bao giờ nhúng `COGNITO_CLIENT_SECRET` vào ứng dụng React SPA phía Client, tại sao việc gửi request xác thực qua NestJS Server là quyết định tối ưu để bảo vệ bí mật hệ thống, và cơ chế tính đồng nhất phối hợp (Coordinated Consistency) giữa Amazon Cognito (`us-east-1`) và cơ sở dữ liệu PostgreSQL (Prisma ORM).

### [Blog 2: Bảo mật xác thực Cognito: SecretHash và thẩm định chữ ký JWT – Phần 2](3.2-Blog2/)

Bài viết tiếp nối Phần 1, tập trung vào hai cơ chế an ninh Backend quan trọng: công thức tính toán `SecretHash` sử dụng thuật toán mã hóa HMAC-SHA256 để chứng minh thẩm quyền với Cognito, mối nguy hiểm nghiêm trọng của việc chỉ `jwt.decode()` đơn thuần, và quy trình thẩm định chữ ký mã hóa RSA của JWT bằng thư viện `aws-jwt-verify` kết hợp tập khóa công khai JWKS.

### [Blog 3: Quản lý Phiên Đăng nhập bằng HttpOnly Cookie, Refresh Token và Phân quyền RBAC kết hợp Cognito Groups](3.3-Blog3/)

Bài viết phân tích phương án lưu trữ JWT trong HttpOnly Signed Cookie chống tấn công XSS/CSRF, quy trình cấp lại token với Refresh Token, và cơ chế phân quyền hai tầng kết hợp giữa NestJS RolesGuard và Cognito User Pool Group `ADMIN`.
