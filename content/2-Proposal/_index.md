---
title: "Proposal"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Startups Blogs - Business Investment Connection Platform
## Investment Connection & Promotion Platform for SMEs and Startups

---

### 1. Executive Summary

**Startups Blogs** is a modern web application platform developed to bridge the connection gap between **Small and Medium Enterprises (SMEs)**, **Business Owners**, **Startups**, and **Investors**.

The system enables businesses to establish standardized capability profiles, showcase financial highlights, and publish capital raising needs. Concurrently, investors can easily search, filter by specialized criteria (industry, funding range, development stage), and initiate direct contact requests with business founders.

From an infrastructure perspective, the entire system is built on a modern enterprise tech stack including **React 19 (Vite, TypeScript)** on the Frontend, **NestJS (REST API, TypeScript)** and **Prisma ORM** on the Backend, combined with a **PostgreSQL (Amazon RDS)** relational database and **Amazon Cognito** identity authentication. The AWS cloud infrastructure is fully automated using **Terraform (Infrastructure as Code)**.

---

### 2. Problem Statement

The current business investment connection market faces key challenges:

- **Information Fragmentation**: Business profiles and funding requests are scattered across multiple isolated channels, social media, or personal documents, making it difficult to reach suitable investors.
- **Lack of Structured Evaluation Data**: Investors lack a centralized, standardized platform to search, compare, and quickly evaluate financial metrics and funding details.
- **Direct Connection Obstacles**: There is a lack of official communication channels enabling investors to actively reach out to founders based on transparent data.
- **Data Management & Security**: Business information requires secure storage, strict access control, and consistent management across web platforms.

---

### 3. Proposed Solution

**Startups Blogs** provides a centralized solution supporting investment discovery and connection:

#### For Businesses (SMEs & Startups)
- Account registration and email verification via Amazon Cognito OTP.
- Creation and management of standardized business profiles (legal info, industry, financial highlights).
- Publishing funding opportunities (`FundingOpportunity`) with clear lifecycle statuses (`Draft`, `Pending Review`, `Published`).

#### For Investors
- Browsing verified and publicly published business listings.
- Utilizing smart filtering by industry, development stage, and funding amount.
- Saving interested companies (`SavedBusiness`) and following company updates (`BusinessFollow`).
- Submitting direct contact requests (`ContactRequest`) to business owners.

#### For Administrators (Admin & Moderator)
- Moderating newly registered business profiles (`PENDING` → `APPROVED`).
- Reviewing profile modification proposals (`ChangeProposal`).
- Managing users, editorial content, and system authorization.

> 💡 **Business Logic Note**: The platform operates as an **Investment Discovery & Connection** engine. The system does **not** process monetary transactions, online payments, or direct equity transfers on the website.

---

### 4. High-Level System Architecture

The system is designed around a clean multi-tier architecture separating the client application from backend server services:

```text
User Browser (React 19 SPA)
      ↓ HTTPS REST API
NestJS Backend (REST API)
      ↓ Prisma ORM
PostgreSQL Database
```

Authentication & Media Storage Services:

```text
React / NestJS Backend  →  Amazon Cognito User Pool (User Authentication)
NestJS Backend          →  Amazon S3 Bucket (Image & Document Storage)
```

- **User Authentication**: Amazon Cognito manages user authentication. NestJS uses `aws-jwt-verify` to validate Cognito-issued JWT Access Tokens (sent via `Authorization: Bearer <accessToken>`) before protected requests access secure API routes.

![High-level architecture of Startups Blogs](/images/proposal/startups-blogs-simple-architecture.png)

*Figure 1. High-level architecture of the Startups Blogs platform.*


---

### 5. AWS Cloud Architecture

The application infrastructure is deployed on the AWS Cloud platform (Region: `us-east-1`):

#### 1. Frontend Delivery
User browsers fetch the static React 19 web application from **Amazon CloudFront CDN**, backed by static web assets hosted in an **Amazon S3 Frontend Bucket**.

#### 2. API Layer & Application Tier
- Client API requests are routed through **Amazon API Gateway**.
- The **NestJS Backend** server runs on **Amazon EC2** inside an **AWS VPC Public Subnet**.
- NestJS processes business logic, validates DTOs, queries the database via Prisma ORM, and handles media uploads to an **Amazon S3 Media Bucket**.

#### 3. Database Tier
The **Amazon RDS PostgreSQL** database operates in an isolated **Private Subnet**. Port 5432 database traffic is strictly restricted via VPC Security Groups to accept connections exclusively from the EC2 Backend.

#### 4. Authentication Service
The **Amazon Cognito User Pool** is configured as a Public App Client (`generate_secret = false`). NestJS receives JWT Access Tokens via the HTTP `Authorization: Bearer` header and cryptographically verifies RSA digital signatures directly against the Cognito JWKS endpoint.

#### 5. Monitoring & Infrastructure Management
- **Amazon CloudWatch**: Monitors operational metrics (such as EC2 CPU Utilization) and configures system alerts.
- **Terraform**: 100% of AWS cloud resources are defined and provisioned automatically via Terraform IaC code (`terraform/`).

![AWS architecture of Startups Blogs](/images/proposal/startups-blogs-aws-architecture.png)

*Figure 2. AWS deployment architecture of the Startups Blogs platform.*


---

### 6. AWS Services Used

| AWS Service | Purpose |
| --- | --- |
| **Amazon S3** | Frontend static web asset hosting and media document storage |
| **Amazon CloudFront** | Content Delivery Network (CDN) for fast frontend page delivery |
| **Amazon API Gateway** | API routing entry point for backend REST API calls |
| **Amazon EC2** | Compute server hosting the NestJS Backend application |
| **Amazon RDS PostgreSQL** | Relational database storing user and business data |
| **Amazon Cognito** | Cloud-Native user identity management and authentication |
| **Amazon CloudWatch** | Infrastructure monitoring metrics and performance alarms |
| **AWS IAM** | Secure access control management between AWS services |
| **Amazon VPC** | Virtual network isolating Public and Private Subnets |
| **Terraform** | Infrastructure as Code (IaC) automation tool |

---

### 7. Technical Implementation

- **Frontend**: Built using React 19 with Vite and TypeScript, managed with Zustand central state stores, handling session tokens automatically via Axios Interceptors.
- **Backend**: Built as a modular NestJS REST API using Prisma ORM to interact with PostgreSQL, protecting routes via `JwtAuthGuard` (`aws-jwt-verify`) and `RolesGuard`.
- **Database & Cloud**: Uses Amazon RDS PostgreSQL for relational data. AWS cloud infrastructure is fully provisioned using Terraform in `terraform/`.

---

### 8. Implementation Roadmap

The project execution is organized into an 8-week internship timeline:

- **Week 1**: AWS fundamentals, business requirements analysis, and Startups Blogs project scoping.
- **Week 2**: Architecture design, AWS VPC network planning, and PostgreSQL schema modeling.
- **Week 3**: Database preparation with Prisma ORM, S3 bucket creation, and Amazon Cognito setup.
- **Week 4**: NestJS Backend API development, business logic execution, and auth route protection.
- **Week 5**: React 19 Frontend development, API integration, and user flow completion.
- **Week 6**: AWS cloud infrastructure provisioning with Terraform, deploying EC2, RDS, and CloudFront.
- **Week 7**: System integration testing, performance optimization, and security audits.
- **Week 8**: Internship report finalization, technical documentation, and final presentation.

---

### 9. Risk Assessment

| Technical / Operational Risk | Severity | Technical Mitigation |
| --- | :---: | --- |
| Infrastructure configuration drift | **Medium** | Automates 100% of AWS infrastructure using version-controlled Terraform IaC |
| JWT Token expiration or auth errors | **Medium** | Uses official `aws-jwt-verify` package and handles client-side Refresh Tokens |
| RDS database connection timeouts | **Low** | Configures tight VPC Security Groups and isolates RDS inside a Private Subnet |
| Unexpected AWS cloud costs | **Low** | Utilizes AWS Free Tier resources and selects appropriate instance sizes (`t3.micro`) |
| Inaccurate business profile data | **Medium** | Implements Admin profile moderation (`PENDING` → `APPROVED`) and `ChangeProposal` diffs |

---

### 10. Expected Outcomes

1. Successfully build a standardized investment connection web platform for SMEs and Startups.
2. Provide transparent tools for business capability profiles and funding opportunity listings.
3. Master full-stack deployment (React 19, NestJS, PostgreSQL) on AWS Cloud infrastructure using Terraform IaC.
4. Integrate Cloud-Native Amazon Cognito authentication into a modern web application architecture.