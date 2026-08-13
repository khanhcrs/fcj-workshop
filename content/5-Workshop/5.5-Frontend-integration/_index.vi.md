---
title: "Tích hợp NestJS REST API, Upload S3 & Thẩm định RSA JWT"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

### 1. Tổng quan (Overview)

Máy chủ **NestJS Backend** đóng vai trò xử lý các yêu cầu **REST API**, tải tập tin đa phương tiện lên **Amazon S3**, thẩm định chữ ký số **RSA** của **JWT** bằng thư viện **`aws-jwt-verify`**, và thực thi phân quyền truy cập đa tầng.

---

### 2. Mục tiêu bài học (Objectives)

- Hiểu rõ cấu trúc các Controller và Service trong ứng dụng NestJS.
- Triển khai cơ chế xác thực **JWT** bằng **`aws-jwt-verify`** kiểm tra với **JWKS** **`us-east-1`**.
- Cấu hình API tải ảnh `POST /upload` sử dụng **`@aws-sdk/client-s3`**.
- Sử dụng **Swagger UI** để thử nghiệm và tài liệu hóa các endpoint **REST API**.

---

### 3. Hệ thống REST API của NestJS Backend

Mã nguồn NestJS tổ chức theo các Module chính:

- **Businesses (`BusinessesController`)**:
  - `GET /businesses`: Tìm kiếm và danh sách doanh nghiệp đã được duyệt (`isVerified: true`, `status: APPROVED`).
  - `POST /businesses`: Tạo hồ sơ doanh nghiệp mới (Yêu cầu **`JwtAuthGuard`**).
  - `PUT /businesses/:id`: Cập nhật thông tin doanh nghiệp (Kiểm tra `business.ownerId === user.userId`).
  - `PUT /businesses/admin/:id/status`: Admin phê duyệt hồ sơ (**`PENDING`** → **`APPROVED`**).
- **Funding Opportunities (`FundingOpportunitiesController`)**:
  - `POST /businesses/:businessId/funding-opportunities`: Đăng tin gọi vốn.
  - `GET /businesses/:businessId/funding-opportunities`: Lấy danh sách tin gọi vốn.
- **Upload (`UploadController`)**:
  - `POST /upload`: Tải file ảnh (logo, cover, avatar) lên **Amazon S3 Media Bucket** / **MinIO** bằng **`@aws-sdk/client-s3`**.
- **Change Proposals (`ProposalsController`)**:
  - `POST /proposals`: Tạo đề xuất thay đổi hồ sơ dưới dạng JSON diff.
  - `POST /proposals/:id/decision`: Admin phê duyệt (**`APPROVED`**) áp dụng diff vào **PostgreSQL** qua `prisma.$transaction`.

---

### 4. Thẩm định Chữ ký số RSA JWT với `aws-jwt-verify`

Phía Backend, **`JwtAuthGuard`** trích xuất Token từ Header **`Authorization: Bearer <accessToken>`** và thẩm định chữ ký số **RSA** thông qua thư viện **`aws-jwt-verify`**:

```typescript
// Trích xuất từ backend/src/auth/guards/jwt-auth.guard.ts
import { CognitoJwtVerifier } from 'aws-jwt-verify';

@Injectable()
export class JwtAuthGuard implements CanActivate {
  private readonly verifier = CognitoJwtVerifier.create({
    userPoolId: process.env.COGNITO_USER_POOL_ID!,
    clientId: process.env.COGNITO_CLIENT_ID!,
    tokenUse: 'access',
  });

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const authorization = request.headers.authorization;
    const token = authorization?.startsWith('Bearer ') ? authorization.slice(7) : undefined;

    if (!token) throw new UnauthorizedException('Missing access token');

    // Thẩm định chữ ký RSA trực tiếp với Cognito JWKS endpoint
    const payload = await this.verifier.verify(token);
    
    // Tự động đồng bộ tài khoản người dùng vào PostgreSQL
    const user = await this.usersService.findOrCreateFromCognito({
      cognitoSub: payload.sub,
      email: payload.email,
      name: payload.name,
    });

    request.user = { userId: user.id, role: user.role };
    return true;
  }
}
```

---

### 5. Giao diện Tài liệu hóa Swagger REST API

NestJS tích hợp tự động module `@nestjs/swagger` giúp tài liệu hóa và kiểm thử trực tiếp các endpoint:

![Giao diện Swagger API](/images/workshop/swagger-api.png)

*Hình 10. Giao diện Swagger dùng để kiểm thử và tài liệu hóa NestJS REST API.*

---

### 6. Xử lý Mã Lỗi HTTP Phổ biến (Troubleshooting)

- **Lỗi HTTP 401 Unauthorized**:
  - Thiếu Header **`Authorization: Bearer <token>`**.
  - Token đã hết hạn hoặc không khớp Cognito User Pool ID / Client ID.
- **Lỗi HTTP 403 Forbidden**:
  - Người dùng không có vai trò **`ADMIN`** khi truy cập các route quản trị (**`RolesGuard`**).
  - Người dùng không phải là chủ sở hữu (`business.ownerId !== userId`) khi sửa/xóa doanh nghiệp.

---

### 7. Tóm tắt (Summary)

Chúng ta đã tích hợp thành công **NestJS REST API** với **`aws-jwt-verify`** để thẩm định chữ ký số **RSA** từ **Amazon Cognito** và cấu hình upload S3. Ở phần tiếp theo (5.6), chúng ta sẽ tích hợp giao diện **React 19 Frontend** và **Admin Dashboard**.
