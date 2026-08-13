---
title: "Workshop Overview & AWS System Architecture"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

### 1. Project Overview

The objective of the **Startups Blogs** project is to build a centralized enterprise web application platform that promotes business capabilities and connects funding opportunities between **Small and Medium Enterprises (SMEs)**, **Business Owners**, **Startups**, and **Investors**.

The system operates on an **Investment Discovery & Connection** model:
- **Businesses**: Register accounts, publish legal information, financial highlights, and capital raising listings (**`FundingOpportunity`**).
- **Investors**: Browse and filter companies by industry/stage, follow updates, and submit direct contact requests (**`ContactRequest`**).
- **Administrators (Admin & Moderator)**: Moderate new business registrations (**`PENDING`** → **`APPROVED`**) and review profile change proposals (**`ChangeProposal`**).

> 💡 **Business Logic Note**: The platform does **not** process monetary transactions, online payments, or direct equity transfers on the website.

---

### 2. Learning Objectives

- Master the overall architecture of a **Full-Stack** application deployed on **AWS Cloud**.
- Understand the role of each tech stack layer: **React 19 Frontend**, **NestJS REST API**, **Prisma ORM**, **PostgreSQL**, and **Amazon Cognito**.
- Map out the cloud infrastructure automated using **Terraform** IaC code.

---

### 3. System Architecture & Data Flow

The core system data flow is structured across application tiers:

```text
User Browser (React 19 SPA)
      ↓ HTTPS REST API
Amazon API Gateway (HTTP API)
      ↓ EC2 Security Group (Port 3000)
NestJS Backend (Amazon EC2 inside Public Subnet)
      ↓ Prisma ORM Client
Amazon RDS PostgreSQL (Private Subnet - Port 5432)
```

Supporting AWS Cloud Services:
- **Amazon Cognito User Pool**: Manages registration, authentication, and 6-digit email OTP verification.
- **Amazon S3** & **Amazon CloudFront**: Hosts static web assets and stores uploaded media documents (`POST /upload`).
- **Amazon CloudWatch** & **Amazon SNS**: Monitors **Amazon EC2** CPU metrics and dispatches email alert notifications.

---

### 4. AWS Cloud Deployment Architecture

![AWS Architecture](/images/workshop/aws-architecture.png)

*Figure 1. Overall AWS Cloud deployment architecture of Startups Blogs.*

> **Note:** The diagram provides a high-level architectural view. The current Terraform configuration does not deploy Route 53 or Multi-AZ RDS.

---

### 5. Application Discovery Interface

![Homepage](/images/workshop/frontend-homepage.png)

*Figure 2. Startups Blogs platform homepage.*

---

### 6. Summary

In this module, we covered the business purpose of **Startups Blogs**, clarified user role boundaries, and visualized the multi-tier application architecture connecting **React 19**, **NestJS Backend**, **Amazon RDS PostgreSQL**, and **AWS** Cloud services. In the next module (5.2), we will set up the local development environment using **Docker Compose**.