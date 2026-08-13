---
title: "Bản đề xuất"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Startups Blogs - Business Investment Connection Platform
## Nền tảng Kết nối Đầu tư và Quảng bá Doanh nghiệp vừa và nhỏ

---

### 1. Tóm tắt điều hành (Executive Summary)

**Startups Blogs** là một nền tảng ứng dụng web hiện đại được phát triển nhằm thu hẹp khoảng cách kết nối giữa các **Doanh nghiệp vừa và nhỏ (SMEs)**, **Chủ doanh nghiệp (Business Owners)**, **Doanh nghiệp khởi nghiệp (Startups)** và các **Nhà đầu tư (Investors)**.

Hệ thống cho phép các doanh nghiệp tạo lập hồ sơ năng lực pháp lý chuẩn hóa, minh bạch hóa điểm tin tài chính và công bố các nhu cầu gọi vốn. Đồng thời, các nhà đầu tư có thể dễ dàng tìm kiếm, lọc theo tiêu chí chuyên sâu (ngành nghề, quy mô vốn, giai đoạn phát triển) và trực tiếp gửi yêu cầu liên hệ tới các nhà sáng lập.

Về mặt hạ tầng, toàn bộ hệ thống được xây dựng trên bộ công nghệ Enterprise modern bao gồm **React 19 (Vite, TypeScript)** ở Frontend, **NestJS (REST API, TypeScript)** và **Prisma ORM** ở Backend, kết hợp với cơ sở dữ liệu quan hệ **PostgreSQL (Amazon RDS)** và dịch vụ xác thực **Amazon Cognito**. Hạ tầng điện toán đám mây AWS được tự động hóa hoàn toàn bằng mã nguồn **Terraform (Infrastructure as Code)**.

---

### 2. Tuyên bố vấn đề (Problem Statement)

Thực trạng thị trường kết nối đầu tư cho doanh nghiệp nhỏ và vừa hiện nay gặp phải các thách thức chính:

- **Phân mảnh thông tin**: Thông tin doanh nghiệp và nhu cầu gọi vốn thường bị phân mảnh trên nhiều kênh riêng lẻ, mạng xã hội hoặc tài liệu cá nhân, gây khó khăn cho việc tiếp cận nhà đầu tư phù hợp.
- **Thiếu cấu trúc dữ liệu đánh giá**: Nhà đầu tư thiếu một nền tảng tập trung, chuẩn hóa để tra cứu, so sánh và thẩm định nhanh các chỉ số tài chính và thông tin gọi vốn.
- **Khó khăn trong kết nối trực tiếp**: Chưa có kênh liên lạc chính thức giúp nhà đầu tư chủ động tiếp cận nhà sáng lập dựa trên các thông tin minh bạch.
- **Quản lý dữ liệu & Bảo mật**: Thông tin doanh nghiệp cần được lưu trữ an toàn, kiểm soát quyền truy cập nghiêm ngặt và duy trì tính nhất quán trên nền tảng web.

---

### 3. Giải pháp đề xuất (Proposed Solution)

**Startups Blogs** cung cấp giải pháp tập trung hỗ trợ kết nối và tìm kiếm cơ hội đầu tư:

#### Đối với Doanh nghiệp (SMEs & Startups)
- Đăng ký tài khoản và xác thực qua mã OTP Email gửi từ Amazon Cognito.
- Tạo lập và quản lý hồ sơ doanh nghiệp chuẩn hóa (thông tin pháp lý, ngành nghề, chỉ số tài chính).
- Đăng tin và công bố các cơ hội gọi vốn (`FundingOpportunity`) với các trạng thái vòng đời rõ ràng (`Draft`, `Pending Review`, `Published`).

#### Đối với Nhà đầu tư (Investors)
- Tra cứu danh sách doanh nghiệp đã được duyệt và đăng tải công khai.
- Sử dụng bộ lọc thông minh theo ngành nghề, giai đoạn phát triển và hạn mức vốn.
- Lưu doanh nghiệp quan tâm (`SavedBusiness`) và theo dõi tin tức doanh nghiệp (`BusinessFollow`).
- Gửi yêu cầu liên hệ trực tiếp (`ContactRequest`) tới chủ doanh nghiệp.

#### Đối với Quản trị viên (Admin & Moderator)
- Phê duyệt hồ sơ doanh nghiệp mới đăng ký (`PENDING` → `APPROVED`).
- Phê duyệt các đề xuất thay đổi thông tin hồ sơ (`ChangeProposal`).
- Quản lý người dùng, nội dung bài viết và phân quyền hệ thống.

> 💡 **Lưu ý nghiệp vụ**: Nền tảng đóng vai trò **kết nối và hỗ trợ tìm kiếm đầu tư (Investment Discovery & Connection)**. Hệ thống **không** thực thi các giao dịch chuyển tiền, thanh toán trực tuyến hay xử lý cổ phần trực tiếp trên trang.

---

### 4. Kiến trúc tổng quan hệ thống (High-Level System Architecture)

Hệ thống được thiết kế theo kiến trúc phân tầng, tách biệt giữa giao diện trình duyệt và các dịch vụ phía server:

```text
User Browser (React 19 SPA)
      ↓ HTTPS REST API
NestJS Backend (REST API)
      ↓ Prisma ORM
PostgreSQL Database
```

Dịch vụ Xác thực & Lưu trữ Đa phương tiện:

```text
React / NestJS Backend  →  Amazon Cognito User Pool (Xác thực người dùng)
NestJS Backend          →  Amazon S3 Bucket (Lưu trữ ảnh & tài liệu)
```

- **Xác thực người dùng**: Amazon Cognito chịu trách nhiệm xác thực người dùng. NestJS sử dụng thư viện `aws-jwt-verify` để thẩm định JWT Access Token (gửi qua `Authorization: Bearer <accessToken>`) trước khi cho phép request truy cập các API bảo mật.

![Kiến trúc tổng quan hệ thống Startups Blogs](/images/proposal/startups-blogs-simple-architecture.png)

*Hình 1. Kiến trúc tổng quan của hệ thống Startups Blogs.*


---

### 5. Kiến trúc AWS Cloud (AWS Cloud Architecture)

Hạ tầng ứng dụng được triển khai trên nền tảng Điện toán Đám mây AWS (Region: `us-east-1`):

#### 1. Phân phối Giao diện (Frontend Delivery)
Trình duyệt người dùng tải giao diện tĩnh React 19 từ **Amazon CloudFront CDN**, giúp tối ưu hóa tốc độ tải trang từ bộ lưu trữ **Amazon S3 Frontend Bucket**.

#### 2. Định tuyến API & Tầng Xử lý (API Layer & Application Tier)
- Requests API từ ứng dụng React được định tuyến qua **Amazon API Gateway**.
- Máy chủ **NestJS Backend** vận hành trên **Amazon EC2** đặt trong **Public Subnet** thuộc **AWS VPC**.
- NestJS xử lý logic nghiệp vụ, kiểm tra DTO, truy vấn cơ sở dữ liệu qua Prisma ORM và xử lý tải ảnh lên **Amazon S3 Media Bucket**.

#### 3. Tầng Cơ sở Dữ liệu (Database Tier)
Cơ sở dữ liệu **Amazon RDS PostgreSQL** đặt trong **Private Subnet** biệt lập. Truy cập cổng 5432 được giới hạn nghiêm ngặt thông qua VPC Security Groups chỉ cho phép kết nối từ EC2 Backend.

#### 4. Dịch vụ Xác thực (Authentication)
**Amazon Cognito User Pool** được cấu hình dạng Public App Client (`generate_secret = false`). NestJS nhận JWT Access Token qua HTTP Header `Authorization: Bearer` và thẩm định chữ ký số RSA trực tiếp với Cognito JWKS endpoint.

#### 5. Giám sát & Quản lý Hạ tầng (Monitoring & IaC)
- **Amazon CloudWatch**: Giám sát các chỉ số vận hành (như EC2 CPU Utilization) và thiết lập cảnh báo hệ thống.
- **Terraform**: 100% tài nguyên đám mây được khai báo và quản lý tự động bằng mã nguồn Terraform (`terraform/`).

![Kiến trúc AWS của Startups Blogs](/images/proposal/startups-blogs-aws-architecture.png)

*Hình 2. Kiến trúc triển khai hệ thống Startups Blogs trên AWS.*


---

### 6. Các dịch vụ AWS sử dụng (AWS Services Used)

| Dịch vụ AWS | Mục đích sử dụng |
| --- | --- |
| **Amazon S3** | Lưu trữ mã nguồn tĩnh Frontend và tài liệu/hình ảnh đa phương tiện |
| **Amazon CloudFront** | Mạng phân phối nội dung (CDN) giúp tăng tốc tải trang Frontend |
| **Amazon API Gateway** | Cửa ngõ định tuyến các yêu cầu REST API tới Backend |
| **Amazon EC2** | Máy chủ điện toán vận hành ứng dụng NestJS Backend |
| **Amazon RDS PostgreSQL** | Cơ sở dữ liệu quan hệ lưu trữ dữ liệu người dùng và doanh nghiệp |
| **Amazon Cognito** | Dịch vụ quản lý danh tính và xác thực người dùng Cloud-Native |
| **Amazon CloudWatch** | Giám sát hiệu năng hạ tầng và thiết lập cảnh báo |
| **AWS IAM** | Quản lý quyền truy cập an toàn giữa các dịch vụ AWS |
| **Amazon VPC** | Mạng nội bộ ảo phân chia Public/Private Subnet an toàn |
| **Terraform** | Công cụ Tự động hóa Hạ tầng bằng Mã nguồn (IaC) |

---

### 7. Triển khai kỹ thuật (Technical Implementation)

- **Frontend**: Triển khai ứng dụng React 19 xây dựng bằng Vite và TypeScript, quản lý trạng thái tập trung với Zustand, tự động xử lý token bằng Axios Interceptors.
- **Backend**: Triển khai NestJS REST API modular, tích hợp Prisma ORM để tương tác với PostgreSQL, bảo vệ API với `JwtAuthGuard` (`aws-jwt-verify`) và `RolesGuard`.
- **Cơ sở Dữ liệu & Đám mây**: Sử dụng Amazon RDS PostgreSQL quản lý dữ liệu quan hệ. Hạ tầng đám mây được khởi tạo hoàn toàn bằng Terraform tại thư mục `terraform/`.

---

### 8. Lộ trình triển khai (Implementation Roadmap)

Quá trình thực hiện dự án được chia thành 8 tuần theo lộ trình thực tập:

- **Tuần 1**: Tìm hiểu tổng quan AWS, phân tích yêu cầu nghiệp vụ và khảo sát dự án Startups Blogs.
- **Tuần 2**: Thiết kế kiến trúc hệ thống, quy hoạch mạng AWS VPC và mô hình hóa dữ liệu PostgreSQL.
- **Tuần 3**: Chuẩn bị cơ sở dữ liệu Prisma ORM, dịch vụ lưu trữ S3 và cấu hình xác thực Amazon Cognito.
- **Tuần 4**: Xây dựng NestJS Backend API, triển khai logic nghiệp vụ và bọc các endpoint xác thực.
- **Tuần 5**: Triển khai giao diện React 19 Frontend, kết nối API và hoàn thiện các trang chức năng.
- **Tuần 6**: Khởi tạo hạ tầng AWS bằng Terraform, đóng gói và triển khai ứng dụng lên EC2, RDS và CloudFront.
- **Tuần 7**: Kiểm thử tích hợp hệ thống, tối ưu hiệu năng và rà soát bảo mật.
- **Tuần 8**: Hoàn thiện báo cáo thực tập, nghiệm thu tài liệu kỹ thuật và báo cáo tổng kết.

---

### 9. Đánh giá rủi ro & Biện pháp giảm thiểu (Risk Assessment)

| Rủi ro kỹ thuật / Vận hành | Mức độ | Biện pháp giảm thiểu |
| --- | :---: | --- |
| Sai lệch cấu hình hạ tầng Đám mây | **Trung bình** | Tự động hóa 100% việc tạo hạ tầng bằng mã nguồn Terraform |
| Lỗi xác thực hoặc hết hạn JWT Token | **Trung bình** | Sử dụng thư viện `aws-jwt-verify` chính thức và xử lý Refresh Token tại Client |
| Gián đoạn kết nối cơ sở dữ liệu RDS | **Thấp** | Cấu hình Security Groups chặt chẽ và đặt RDS trong Private Subnet an toàn |
| Chi phí AWS vượt ngân sách dự kiến | **Thấp** | Tận dụng gói AWS Free Tier, chọn kích thước EC2/RDS (`t3.micro`) phù hợp |
| Thông tin doanh nghiệp chưa chính xác | **Trung bình** | Triển khai luồng duyệt Admin (`PENDING` → `APPROVED`) và cơ chế `ChangeProposal` |

---

### 10. Kết quả kỳ vọng (Expected Outcomes)

1. Xây dựng thành công nền tảng web kết nối đầu tư chuẩn hóa cho các Doanh nghiệp vừa và nhỏ (SMEs) và Startups.
2. Cung cấp công cụ minh bạch hóa thông tin năng lực doanh nghiệp và các cơ hội gọi vốn.
3. Làm chủ quy trình triển khai ứng dụng Full-Stack (React 19, NestJS, PostgreSQL) trên hạ tầng Đám mây AWS bằng Terraform IaC.
4. Tích hợp thành công dịch vụ xác thực Cloud-Native Amazon Cognito vào mô hình ứng dụng web hiện đại.