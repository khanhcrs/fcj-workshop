---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

#### Overview

This workshop provides a **7-module step-by-step hands-on guide** to designing, building, and automating a full enterprise cloud infrastructure on **AWS** (Region: **`us-east-1`**) for the **Startups Blogs** platform.

The system combines a full-stack architecture comprising **React 19 (TypeScript, Vite)** on the frontend, **NestJS REST API** on the backend, **Amazon RDS PostgreSQL**, **Amazon S3** media storage, **Amazon CloudFront** CDN, **Amazon API Gateway**, **Amazon Cognito** cloud-native authentication, **Amazon CloudWatch** monitoring, and 100% Infrastructure as Code powered by **Terraform**.

#### Key Technical Highlights

- **Infrastructure as Code via Terraform**: All AWS resources (**Amazon VPC**, **Amazon EC2**, **Amazon RDS**, **Amazon Cognito**, **Amazon API Gateway**, **Amazon S3**, **Amazon CloudFront**, **Amazon CloudWatch**, **Amazon SNS**) are declared inside the `terraform/` directory.
- **Delegated Identity Management**: User passwords are never stored in **PostgreSQL**; credential handling is offloaded entirely to **Amazon Cognito User Pool**.
- **Public App Client Configuration**: Configured as a **Public App Client** (**`generate_secret = false`**) matching modern SPA architecture best practices.
- **Cryptographic RSA JWT Verification via `aws-jwt-verify`**: NestJS backend cryptographically validates **RS256** signatures against **Cognito JWKS** endpoints in **`us-east-1`**.
- **Dual-Layer Authorization Guard**: Integrates **`JwtAuthGuard`** (identity verification), **`RolesGuard`** (system roles **`ADMIN`**, **`MODERATOR`**, **`USER`**), and service-level **Resource Ownership** checks (**`ownerId === userId`**).
- **S3 Media Uploads (`POST /upload`)**: Uses **`@aws-sdk/client-s3`** for uploading business logos, avatars, and cover images to **Amazon S3 Media Bucket** / **MinIO**.

#### Workshop Modules

1. [5.1 Workshop Overview & AWS System Architecture](5.1-Workshop-overview/)
2. [5.2 Local Development Environment, Docker & PostgreSQL Setup](5.2-Prerequiste/)
3. [5.3 Database Schema Initialization, Prisma ORM & Seed Data](5.3-Cognito-setup/)
4. [5.4 Amazon Cognito User Pool, Public App Client & RBAC Configuration](5.4-Backend-integration/)
5. [5.5 NestJS REST API Integration, S3 Uploads & RSA JWT Verification](5.5-Frontend-integration/)
6. [5.6 React 19 Frontend Integration, Business Discovery & Admin Dashboard](5.6-Security-review/)
7. [5.7 Terraform Infrastructure Automation, Monitoring & Resource Cleanup](5.7-Cleanup/)