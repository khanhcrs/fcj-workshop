---
title: "Tích hợp React 19 Frontend, Tìm kiếm Doanh nghiệp & Admin Dashboard"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

### 1. Tổng quan (Overview)

Ứng dụng **React 19 Frontend** (Vite + TypeScript) cung cấp giao diện người dùng hiện đại, kết nối tới **NestJS REST API**, quản lý trạng thái phiên làm việc với **Zustand**, tự động gắn Bearer Token qua **Axios Interceptors**, và cung cấp các luồng giao diện cho **Doanh nghiệp**, **Nhà đầu tư**, và **Quản trị viên (Admin Dashboard)**.

---

### 2. Mục tiêu bài học (Objectives)

- Hiểu cách tổ chức trạng thái xác thực phía Client bằng Zustand (`authStore.ts`).
- Cấu hình **Axios Request Interceptors** tự động đính kèm **`Authorization: Bearer <accessToken>`**.
- Khai thác giao diện khám phá doanh nghiệp, lọc thông minh và xem tin gọi vốn.
- Hiểu quy trình vận hành, kiểm soát truy cập **RBAC** và xử lý lỗi xác thực trên **Admin Dashboard UI** (`/admin/*`).

---

### 3. Quản lý Trạng thái & Interceptor (`authStore` & Axios)

Sau khi đăng nhập thành công với Cognito, Frontend lưu Access Token vào `localStorage` và tự động gắn vào các request HTTP:

```typescript
// Cấu hình Axios Request Interceptor trong frontend/src/lib/api.ts
import axios from 'axios';
import { useAuthStore } from '@/stores/authStore';

export const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL || '/api',
});

api.interceptors.request.use((config) => {
  const token = useAuthStore.getState().token || localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});
```

---

### 4. Giao diện Khám phá Doanh nghiệp cho Nhà đầu tư

Nhà đầu tư sử dụng giao diện web để tra cứu doanh nghiệp và tìm kiếm cơ hội gọi vốn:
- Lọc doanh nghiệp theo ngành nghề, địa điểm, quy mô nhân sự và giai đoạn phát triển.
- Xem điểm tin tài chính (`financialHighlights`) và lịch sử gọi vốn (`FundingRound`).
- Lưu doanh nghiệp (**`SavedBusiness`**), theo dõi (**`BusinessFollow`**) và gửi yêu cầu liên hệ (**`ContactRequest`**).

![Danh sách doanh nghiệp](/images/workshop/frontend-business-listing.png)

*Hình 11. Giao diện danh sách và khám phá doanh nghiệp trên Startups Blogs.*

---

### 5. Kiểm soát truy cập theo vai trò trên Frontend

Khi người dùng đã đăng nhập thành công nhưng không có quyền quản trị viên (không thuộc Cognito Group **`ADMIN`**), hệ thống Frontend sẽ ngăn chặn truy cập vào các tuyến đường quản trị (`/admin/*`) và hiển thị trang thông báo từ chối truy cập (**Access Denied**):

![Admin Access Denied](/images/workshop/admin-dashboard-dennied.png)

*Hình 12. Trang Access Denied được hiển thị khi người dùng không có quyền truy cập vào chức năng quản trị.*

> **Lưu ý:** Việc bảo vệ route ở Frontend giúp kiểm soát điều hướng và trải nghiệm người dùng, nhưng Backend vẫn phải thực hiện phân quyền bằng **`JwtAuthGuard`**, **`RolesGuard`** và kiểm tra **Resource Ownership**.

---

### 6. Xử lý sự cố Tích hợp Session Admin (Troubleshooting)

Trong trường hợp kết nối từ **Admin Dashboard** tới **Amazon API Gateway** hoặc **NestJS Backend** bị gián đoạn hoặc không xác minh được phiên đăng nhập (do Token hết hạn hoặc lỗi kết nối cổng Gateway), hệ thống hiển thị thông báo sự cố:

![Lỗi xác minh phiên Admin](/images/workshop/admin-dashboard.png)

*Hình 13. Ví dụ lỗi xác minh phiên Admin trong quá trình tích hợp Backend/API Gateway.*

---

### 7. Tóm tắt (Summary)

Chúng ta đã tích hợp thành công giao diện **React 19 Frontend** với **NestJS Backend**, xử lý tự động Bearer Token và quản lý luồng UI cho cả Nhà đầu tư và Quản trị viên. Ở bài thực hành cuối cùng (5.7), chúng ta sẽ tiến hành tự động hóa hạ tầng đám mây bằng **Terraform** và thực thi dọn dẹp tài nguyên.
