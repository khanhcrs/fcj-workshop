---
title: "Cấu hình Hạ tầng AWS: VPC, Amazon RDS PostgreSQL & Triển khai Backend EC2"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

### Tổng quan (Overview)

Trong mô hình ứng dụng **Startups Blogs**, việc xây dựng một hạ tầng điện toán đám mây an toàn và có khả năng mở rộng là điều kiện tiên quyết. Module này hướng dẫn chi tiết quy trình khởi tạo mạng ảo riêng **Amazon VPC**, phân hoạch **Subnets**, thiết lập cổng **Internet Gateway**, cấu hình **Security Groups**, triển khai cơ sở dữ liệu **Amazon RDS PostgreSQL**, và vận hành ứng dụng **NestJS Backend** 24/7 trên máy chủ **Amazon EC2**.

---

## 5.3.1 VPC, Subnets, Internet Gateway & Security Groups

#### 1. Tổng quan (Overview)
Tầng mạng đóng vai trò thiết lập ranh giới bảo mật cho toàn bộ tài nguyên hệ thống. Mạng ảo riêng **Amazon VPC** giúp cô lập tài nguyên máy chủ và cơ sở dữ liệu khỏi các truy cập trái phép từ Internet.

#### 2. Mục tiêu bài học (Objectives)
- Khởi tạo Amazon VPC **`Startups-Blogs-vpc`** với dải mạng CIDR **`10.0.0.0/16`**.
- Lập lịch địa chỉ các phân mạng Subnets: **`10.0.0.0/20`**, **`10.0.128.0/20`**, và **`10.0.200.0/24`**.
- Tạo cổng **Internet Gateway** **`Startups-Blogs-igw`** cho phép lưu lượng mạng từ Public Subnet ra ngoài Internet.
- Cấu hình quy tắc tường lửa cho **EC2 Security Group** và **RDS Security Group**.

#### 3. Thực hành từng bước (Step-by-Step Implementation)

##### Bước 1: Khởi tạo Amazon VPC
Truy cập AWS Console -> Chọn dịch vụ **VPC** -> Đặt tên VPC: **`Startups-Blogs-vpc`** và nhập IPv4 CIDR: **`10.0.0.0/16`**.

![Tạo VPC](/images/workshop/5.3/5.3.1-step1-create-vpc.png)

*Hình 1. Giao diện khởi tạo Amazon VPC cho dự án Startups Blogs.*

##### Bước 2: Chia phân mạng Subnets
Tạo các phân mạng Subnets trong VPC với các dải địa chỉ:
- Public Subnet: **`10.0.0.0/20`**
- Private Subnet 1: **`10.0.128.0/20`**
- Private Subnet 2: **`10.0.200.0/24`**

![Subnets](/images/workshop/5.3/5.3.1-step2-subnets.png)

*Hình 2. Danh sách Subnets trong Amazon VPC.*

##### Bước 3: Tạo và gắn Internet Gateway
Tạo cổng Internet Gateway tên **`Startups-Blogs-igw`** và thực hiện lệnh **Attach to VPC** vào `Startups-Blogs-vpc`.

![Internet Gateway](/images/workshop/5.3/5.3.1-step3-internet-gateway.png)

*Hình 3. Cấu hình Internet Gateway cho phép truy cập Internet.*

##### Bước 4: Cấu hình EC2 Security Group
Tạo Security Group cho máy chủ Backend EC2, thiết lập **Inbound Rules**:
- **SSH (Port 22)**: Cho phép kết nối từ địa chỉ IP của nhà phát triển.
- **Custom TCP (Port 3000)**: Cho phép API Gateway/Frontend gọi vào ứng dụng NestJS.

![EC2 Security Group](/images/workshop/5.3/5.3.1-step4-ec2-security-group.png)

*Hình 4. Cấu hình Inbound Rules cho EC2 Security Group.*

##### Bước 5: Cấu hình RDS Security Group
Tạo Security Group cho cơ sở dữ liệu PostgreSQL, thiết lập **Inbound Rule**:
- **PostgreSQL (Port 5432)**: **Chỉ cho phép** kết nối từ **EC2 Security Group** (giới hạn truy cập nội bộ, không mở Public).

![RDS Security Group](/images/workshop/5.3/5.3.1-step5-rds-security-group.png)

*Hình 5. Cấu hình Inbound Rules cho RDS Security Group giới hạn kết nối từ EC2.*

#### 4. Kết quả kỳ vọng (Expected Result)
Mạng ảo **`Startups-Blogs-vpc`** được kích hoạt với cổng **`Startups-Blogs-igw`** và hai nhóm Security Groups cô lập thành công.

#### 5. Xử lý sự cố (Troubleshooting)
- **Không SSH được vào EC2**: Kiểm tra lại Inbound Rule của EC2 Security Group (Port **`22`**) và bảng tuyến đường **Route Table** đã gán hướng đi `0.0.0.0/0` tới Internet Gateway hay chưa.

---

## 5.3.2 Amazon RDS PostgreSQL Deployment

#### 1. Tổng quan (Overview)
**Amazon RDS PostgreSQL** cung cấp giải pháp quản trị cơ sở dữ liệu quan hệ đám mây, tự động sao lưu và bảo mật cao cho dự án Startups Blogs.

#### 2. Mục tiêu bài học (Objectives)
- Khởi tạo RDS Database Instance tên **`startups-blogs-db`** chạy engine **PostgreSQL** trên loại máy chủ **`db.t4g.micro`**.
- Đặt mật khẩu quản trị và tắt tính năng **Public Access** để bảo vệ dữ liệu.
- Thu thập thông số **RDS Endpoint** và cấu hình biến `DATABASE_URL`.

#### 3. Thực hành từng bước (Step-by-Step Implementation)

##### Bước 1: Lựa chọn Database Engine
Trong dịch vụ Amazon RDS -> Click **Create database** -> Chọn engine **PostgreSQL** (chọn bản **Standard Create**).

![Lựa chọn Engine](/images/workshop/5.3/5.3.2-step1-database-engine.png)

*Hình 6. Bước lựa chọn Engine cho Amazon RDS Database.*

> **Lưu ý:** Màn hình ví dụ hiển thị Aurora PostgreSQL, trong triển khai thực tế của Startups Blogs hãy chọn engine **PostgreSQL** tiêu chuẩn với loại instance **`db.t4g.micro`** và tên DB Identifier: **`startups-blogs-db`**.

##### Bước 2: Thiết lập thông số Database
Nhập **Master Username** và đặt **Master Password** an toàn cho cơ sở dữ liệu.

![Cấu hình Database](/images/workshop/5.3/5.3.2-step2-database-settings.png)

*Hình 7. Thiết lập thông số Master Username và Password cho RDS.*

##### Bước 3: Cấu hình mạng & Bảo mật
Chọn VPC triển khai và cấu hình bảo mật:
- **VPC**: Chọn **`Startups-Blogs-vpc`**.
- **Publicly accessible**: Chọn **No** (tắt kết nối công khai).
- **VPC Security Group**: Chọn RDS Security Group đã tạo ở bài 5.3.1.

![Cấu hình Kết nối](/images/workshop/5.3/5.3.2-step3-database-connectivity.png)

*Hình 8. Cấu hình mạng VPC và bảo mật cho RDS Instance.*

> **Lưu ý:** Màn hình khởi tạo minh họa chọn Default VPC. Đối với bản triển khai chính thức của Startups Blogs, bạn chọn VPC **`Startups-Blogs-vpc`** và gán RDS Security Group đã tạo ở bài 5.3.1.

##### Bước 4: Xác nhận trạng thái Database
Chờ cơ sở dữ liệu khởi tạo hoàn tất cho đến khi cột **Status** chuyển sang **Available**.

![RDS Available](/images/workshop/5.3/5.3.2-step4-rds-available.png)

*Hình 9. Amazon RDS Instance ở trạng thái Available.*

##### Bước 5: Ghi nhận RDS Endpoint
Truy cập chi tiết Database -> Ghi nhận địa chỉ **Endpoint** và **Port** (5432).

![RDS Endpoint](/images/workshop/5.3/5.3.2-step5-rds-endpoint.png)

*Hình 10. Thông tin Endpoint và Port của Amazon RDS PostgreSQL.*

Cấu hình chuỗi kết nối trong file `.env` của NestJS Backend:
```env
DATABASE_URL="postgresql://<username>:<password>@<rds-endpoint>:5432/startups_blogs?schema=public"
```

#### 4. Kết quả kỳ vọng (Expected Result)
Cơ sở dữ liệu **`startups-blogs-db`** đạt trạng thái **Available** và sẵn sàng tiếp nhận kết nối từ EC2 Backend.

#### 5. Xử lý sự cố (Troubleshooting)
- **Lỗi Timeout kết nối từ Backend**: Kiểm tra RDS Security Group đã cho phép Inbound Port **`5432`** từ EC2 Security Group hay chưa.

---

## 5.3.3 Amazon EC2 & NestJS Backend Deployment

#### 1. Tổng quan (Overview)
Máy chủ điện toán **Amazon EC2** đóng vai trò môi trường thực thi ứng dụng **NestJS Backend**, kết nối cơ sở dữ liệu **Amazon RDS PostgreSQL** và dịch vụ xác thực **Amazon Cognito**.

#### 2. Mục tiêu bài học (Objectives)
- Khởi chạy máy chủ EC2 tên **`startups-blogs-backend`** chạy **Ubuntu Server**.
- Cấu hình SSH Key Pair **`startups-key`** để quản trị máy chủ từ xa.
- Kiểm tra môi trường **Node.js v20.20.2**, **npm 10.8.2**, và cài đặt **PM2 7.0.3**.
- Cấu hình file `.env` an toàn và vận hành ứng dụng NestJS bằng PM2.

#### 3. Thực hành từng bước (Step-by-Step Implementation)

##### Bước 1: Khởi chạy máy chủ Amazon EC2
Mở dịch vụ EC2 -> Click **Launch Instance** -> Đặt tên: **`startups-blogs-backend`** -> Chọn OS: **Ubuntu Server**.

![Khởi chạy EC2](/images/workshop/5.3/5.3.3-step1-launch-ec2.png)

*Hình 11. Bước cấu hình tên và loại máy chủ Amazon EC2.*

> **Lưu ý:** Màn hình cấu hình ban đầu hiển thị gợi ý loại instance `t3.micro`. Khi khởi chạy thực tế cho bài test, bạn có thể chọn **`t2.micro`** hoặc **`t3.micro`** tương thích với tài khoản AWS.

##### Bước 2: Cấu hình SSH Key Pair
Tạo mới hoặc chọn SSH Key Pair tên **`startups-key`** để kết nối an toàn vào máy chủ.

![Key Pair](/images/workshop/5.3/5.3.3-step2-key-pair.png)

*Hình 12. Chọn Key Pair SSH cho máy chủ EC2.*

##### Bước 3: Xác nhận trạng thái EC2 Instance
Sau khi Launch thành công, kiểm tra bảng điều khiển EC2 cho đến khi **Instance state** hiển thị **Running**.

![EC2 Running](/images/workshop/5.3/5.3.3-step3-ec2-running.png)

*Hình 13. Máy chủ EC2 ở trạng thái Running.*

##### Bước 4: Kiểm tra môi trường Node.js & PM2
Kết nối SSH vào máy chủ và thực thi kiểm tra phiên bản môi trường:
```bash
node -v
npm -v
pm2 -v
```

![Môi trường Node.js](/images/workshop/5.3/5.3.3-step4-nodejs-pm2.png)

*Hình 14. Kiểm tra phiên bản Node.js v20.20.2, npm 10.8.2 và PM2 7.0.3 trên EC2.*

##### Bước 5: Cấu hình biến môi trường an toàn (`.env`)

<!-- ================================================== -->
<!-- IMAGE PLACEHOLDER: BACKEND_ENV_SANITIZED -->
<!-- ================================================== -->

> **Hình 15. Ví dụ cấu hình an toàn cho các biến môi trường Backend.**

Ví dụ cấu hình mẫu an toàn không chứa secret thực:
```env
PORT=3000
DATABASE_URL="postgresql://postgres:********@startups-blogs-db.xxxx.us-east-1.rds.amazonaws.com:5432/startups_blogs"
COGNITO_USER_POOL_ID=us-east-1_xxxxxxxxx
COGNITO_CLIENT_ID=xxxxxxxxxxxxxxxxxxxxxxxxxx
AWS_S3_BUCKET=startups-blogs-media
AWS_S3_ENDPOINT=https://s3.us-east-1.amazonaws.com
AWS_S3_REGION=us-east-1
AWS_S3_ACCESS_KEY=********
AWS_S3_SECRET_KEY=********
```

> ⚠️ **Cảnh báo bảo mật**: Thông tin `AWS_S3_ACCESS_KEY` và `AWS_S3_SECRET_KEY` là dữ liệu bí mật tuyệt đối. Không bao giờ commit file `.env` chứa chìa khóa thực lên mã nguồn Git public.

##### Bước 6: Vận hành Backend với PM2 & Kiểm thử API
Biên dịch ứng dụng và khởi chạy quy trình bằng **PM2**:
```bash
npm run build
pm2 start dist/main.js --name "startups-blogs-backend"
pm2 status
curl -I http://localhost:3000
```

![PM2 Running](/images/workshop/5.3/5.3.3-step6-pm2-backend-running.png)

*Hình 16. Kiểm tra trạng thái ứng dụng NestJS Backend chạy online với PM2.*

> 💡 **Lưu ý cấu hình CORS**: Khi triển khai lên môi trường Production thực tế, cấu hình CORS của NestJS không nên duy trì giá trị mở `Access-Control-Allow-Origin: *` mà cần giới hạn chính xác domain của Frontend/CloudFront.

#### 4. Kết quả kỳ vọng (Expected Result)
Lệnh `pm2 status` hiển thị ứng dụng `startups-blogs-backend` ở trạng thái **online** và lệnh `curl -I http://localhost:3000` phản hồi **`HTTP/1.1 200 OK`**.

#### 5. Xử lý sự cố (Troubleshooting)
- **PM2 hiển thị trạng thái `errored`**: Kiểm tra file log bằng lệnh `pm2 logs` (nguyên nhân thường do sai kết nối `DATABASE_URL` hoặc thiếu biến môi trường Cognito).

---

## 5.3.4 Amazon S3 & CloudFront cho Phân phối Frontend

#### 1. Tổng quan (Overview)
Bộ lưu trữ **Amazon S3** và mạng phân phối nội dung **Amazon CloudFront** đảm nhận việc lưu trữ các tệp tĩnh mã nguồn **React 19 SPA** cũng như phân phối tập tin đa phương tiện (logo, avatar, ảnh bìa) cho hệ thống **Startups Blogs**.

#### 2. Mục tiêu bài học (Objectives)
- Phân biệt vai trò của các S3 Buckets trong dự án (**`startups-blogs-frontend`** và **`startups-blogs-media`**).
- Cấu hình quy tắc **CORS** cho phép ứng dụng Web tương tác an toàn với S3.
- Cấu hình **Amazon CloudFront Distribution** làm tầng CDN tối ưu tốc độ truy cập.
- Cấu hình quy tắc **Custom Error Pages** (403 & 404 → `/index.html`) hỗ trợ điều hướng client-side **React Router SPA**.

#### 3. Thực hành từng bước (Step-by-Step Implementation)

##### Bước 1: Khởi tạo các Amazon S3 Buckets
Dự án **Startups Blogs** sử dụng các S3 Buckets chính:
- **`startups-blogs-frontend`**: Lưu trữ các tệp tĩnh (HTML, JS, CSS) của ứng dụng **React 19**.
- **`startups-blogs-media`**: Lưu trữ ảnh logo, avatar, và tệp đính kèm do người dùng/doanh nghiệp tải lên (`POST /upload`).

![S3 Media Bucket](/images/workshop/5.3/5.3.4-step1-media-bucket.png)

*Hình 17. Cấu hình thông tin Amazon S3 Bucket lưu trữ đa phương tiện.*

> ⚠️ **Lưu ý bảo mật**: Màn hình khởi tạo thử nghiệm có thể hiển thị tính năng Block Public Access ở trạng thái tắt. Trong môi trường Production thực tế, khuyến nghị bảo vệ các S3 Bucket ở chế độ riêng tư (Private) và chỉ cho phép truy cập thông qua Amazon CloudFront Origin Access Control (OAC).

##### Bước 2: Cấu hình S3 Cross-Origin Resource Sharing (CORS)
Cấu hình JSON CORS cho S3 Bucket cho phép ứng dụng Web gửi các HTTP Requests:

```json
[
  {
    "AllowedHeaders": ["*"],
    "AllowedMethods": ["GET", "PUT", "POST", "DELETE"],
    "AllowedOrigins": ["*"],
    "ExposeHeaders": []
  }
]
```

![S3 CORS](/images/workshop/5.3/5.3.4-step2-cors.png)

*Hình 18. Cấu hình quy tắc S3 CORS cho phép tương tác qua HTTP API.*

> 💡 **Lưu ý triển khai**: Cấu hình `AllowedOrigins: ["*"]` phù hợp cho môi trường thử nghiệm workshop. Khi đưa lên Production, giá trị `AllowedOrigins` cần được giới hạn chính xác domain ứng dụng.

##### Bước 3: Danh sách Amazon S3 Buckets
Kiểm tra danh sách các Buckets đã khởi tạo trên giao diện Amazon S3 Console.

![S3 Buckets List](/images/workshop/5.3/5.3.4-step3-s3-buckets.png)

*Hình 19. Danh sách các Amazon S3 Buckets phục vụ dự án Startups Blogs.*

##### Bước 4: Cấu hình Amazon CloudFront Distribution
Khởi tạo CloudFront Distribution trỏ Origin tới **`startups-blogs-frontend`** S3 Bucket với các thông số:
- **Status**: **Enabled**
- **Type**: **Standard**
- **Origin**: `startups-blogs-frontend.s3.us-east-1.amazonaws.com`

![CloudFront Distribution](/images/workshop/5.3/5.3.4-step5-distribution.png)

*Hình 20. Khởi tạo và kích hoạt Amazon CloudFront CDN Distribution.*

##### Bước 5: Cấu hình Routing cho React SPA (Custom Error Pages)
Ứng dụng **React SPA** xử lý điều hướng ở phía Client (**React Router**). Khi người dùng truy cập trực tiếp các đường dẫn như `/businesses`, `/profile`, hoặc `/admin`, S3 sẽ trả về lỗi HTTP 403 hoặc 404.

Cấu hình **Custom Error Responses** trên CloudFront:
- **HTTP Error Code**: **`403`** & **`404`**
- **Response Page Path**: **`/index.html`**
- **HTTP Response Code**: **`200`**

![CloudFront SPA Error Pages](/images/workshop/5.3/5.3.4-step6-spa-error-pages.png)

*Hình 21. Cấu hình CloudFront Custom Error Pages phục vụ điều hướng React Router SPA.*

#### 4. Kết quả kỳ vọng (Expected Result)
Người dùng truy cập domain CloudFront có thể tải trang React SPA mượt mà và chuyển đổi các route client-side mà không bị lỗi 404.

#### 5. Xử lý sự cố (Troubleshooting)
- **Lỗi 404 khi F5 làm mới trang**: Chưa cấu hình quy tắc Custom Error Responses (403/404 → `/index.html` HTTP 200) trên CloudFront Distribution.

---

## 5.3.5 Tích hợp Amazon API Gateway

#### 1. Tổng quan (Overview)
**Amazon API Gateway** đóng vai trò là điểm truy cập công khai (Public API Entry Point) trung gian giữa ứng dụng **React 19 Frontend** và máy chủ **NestJS Backend** chạy trên EC2.

#### 2. Mục tiêu bài học (Objectives)
- Khởi tạo **HTTP API** tên **`startups-blogs-api`** tại region **`us-east-1`**.
- Tạo tuyến đường proxy **`ANY /{proxy+}`** kết nối tới EC2 Backend Port 3000.
- Cấu hình quy tắc **CORS** cho API Gateway tương thích với kiến trúc **Bearer Token**.
- Thu thập địa chỉ **Invoke URL** để cấu hình biến `VITE_API_URL` phía Frontend.

#### 3. Thực hành từng bước (Step-by-Step Implementation)

##### Bước 1: Khởi tạo Amazon HTTP API
Vào dịch vụ API Gateway -> Chọn **Build HTTP API** -> Đặt tên API: **`startups-blogs-api`** -> Protocol: **HTTP** -> Endpoint Type: **Regional**.

![Tạo API Gateway](/images/workshop/5.3/5.3.5-step1-create-api.png)

*Hình 22. Khởi tạo Amazon API Gateway (HTTP API) cho Startups Blogs.*

##### Bước 2: Cấu hình Backend Integration Proxy
Tạo Integration loại **HTTP URI** trỏ về dịch vụ NestJS Backend trên EC2:
- **Target**: `http://<EC2_PUBLIC_IP>:3000/{proxy}`
- **Timeout**: **`30000`** milliseconds

![Backend Integration](/images/workshop/5.3/5.3.5-step2-integration.png)

*Hình 23. Cấu hình HTTP Proxy Integration chuyển tiếp request tới EC2 Backend.*

> 💡 **Lưu ý kiến trúc**: Kết nối trực tiếp từ API Gateway tới Public IP của EC2 phù hợp cho quy trình kiểm thử workshop. Trong kiến trúc Production nâng cao, nên sử dụng **VPC Link** kết nối tới Private Load Balancer nội bộ.

##### Bước 3: Định tuyến Route `ANY /{proxy+}`
Cấu hình route **`ANY /{proxy+}`** để chuyển tiếp toàn bộ các HTTP Methods (`GET`, `POST`, `PUT`, `DELETE`) và đường dẫn API sang NestJS.

![API Route](/images/workshop/5.3/5.3.5-step3-route.png)

*Hình 24. Định tuyến proxy route ANY /{proxy+} trên API Gateway.*

> 🔐 **Lưu ý về Bảo mật & Xác thực**: Giao diện hiển thị *"No authorizer attached to this route"*. Điều này **không** có nghĩa là ứng dụng thiếu xác thực, mà việc xác thực JWT đang được chuyển giao và thực thi trực tiếp bên trong NestJS Backend thông qua **`JwtAuthGuard`**, **`aws-jwt-verify`** và **Cognito JWKS**. API Gateway đóng vai trò làm tầng định tuyến proxy truyền tải.

##### Bước 4: Cấu hình API Gateway CORS
Cấu hình CORS trên API Gateway phù hợp với kiến trúc **Bearer Token**:
- **AllowedOrigins**: `*`
- **AllowedHeaders**: `*`
- **AllowedMethods**: `GET`, `POST`, `OPTIONS`, `PATCH`, `PUT`, `DELETE`
- **AllowCredentials**: **NO** (khớp với mô hình gửi Token qua Header `Authorization: Bearer <accessToken>`).

![API Gateway CORS](/images/workshop/5.3/5.3.5-step4-cors.png)

*Hình 25. Cấu hình quy tắc API Gateway CORS.*

##### Bước 5: Lấy địa chỉ Invoke URL
Chọn Stage **`$default`** (bật **Auto-deploy**) -> Thu thập địa chỉ **Invoke URL**.

![API Gateway Invoke URL](/images/workshop/5.3/5.3.5-step5-invoke-url.png)

*Hình 26. Địa chỉ Invoke URL chính thức của Amazon API Gateway.*

Cấu hình địa chỉ Invoke URL vào biến môi trường phía React Frontend:
```env
VITE_API_URL=https://<api-id>.execute-api.us-east-1.amazonaws.com
```

#### 4. Kết quả kỳ vọng (Expected Result)
Mọi yêu cầu HTTP API gửi tới API Gateway Invoke URL đều được chuyển tiếp mượt mà tới NestJS Backend trên EC2 và phản hồi dữ liệu thành công.

#### 5. Xử lý sự cố (Troubleshooting)
- **Lỗi 500 Internal Server Error / 503 Service Unavailable**: Kiểm tra lại EC2 Security Group đã mở Port **`3000`** cho API Gateway truy cập hay chưa.

---

### Mô hình Kiến trúc Hạ tầng Hoàn chỉnh (Complete Infrastructure Architecture)

Sau khi hoàn thành các phần từ 5.3.1 đến 5.3.5, hạ tầng đám mây AWS của **Startups Blogs** được vận hành theo mô hình:

```text
Trình duyệt Người dùng (React 19 SPA)
   │
   ├──────► Amazon CloudFront (CDN) ──────► Amazon S3 Frontend Bucket
   │
   └──────► Amazon API Gateway (HTTP API)
               │
               ▼
            Amazon EC2 (NestJS Backend - Port 3000)
               │
               ▼
            Amazon RDS PostgreSQL (Private Subnet - Port 5432)

Hỗ trợ quản lý & lưu trữ:
- Amazon Cognito User Pool (Xác thực danh tính & JWKS us-east-1)
- Amazon S3 Media Bucket (Lưu trữ ảnh & file đính kèm)
```

---

### Tóm tắt (Summary)

Chúng ta đã hoàn thành xuất sắc việc xây dựng và kết nối toàn bộ hạ tầng điện toán đám mây cho dự án **Startups Blogs**: khởi tạo mạng ảo **Amazon VPC**, cơ sở dữ liệu **Amazon RDS PostgreSQL**, máy chủ **Amazon EC2**, lưu trữ **Amazon S3**, mạng phân phối **Amazon CloudFront** và cổng **Amazon API Gateway**. Trong module tiếp theo (5.4), chúng ta sẽ sâu sát vào việc cấu hình **Amazon Cognito User Pool** và phân quyền RBAC.
