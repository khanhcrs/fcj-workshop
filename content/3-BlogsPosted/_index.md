---
title: "BlogsPosted"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

During the **First Cloud AI Journey (FCAJ)** internship program, I authored three detailed technical blog posts exploring **Amazon Cognito Cloud Authentication Architecture**, **JWT Token Security via aws-jwt-verify**, and **Session Management using HttpOnly Cookies & RBAC Authorization for Startups Blogs**.

---

### [Blog 1: Why Should Authentication Requests Go Through the Backend? – Amazon Cognito with React and NestJS](3.1-Blog1/)
This article analyzes why browser-based React SPAs must never contain the confidential `COGNITO_CLIENT_SECRET`, explains why backend-mediated authentication through NestJS protects system secrets, and details coordinated application-level consistency between Amazon Cognito (`us-east-1`) and PostgreSQL (Prisma ORM).

### [Blog 2: Securing Amazon Cognito Authentication: SecretHash and JWT Signature Verification – Part 2](3.2-Blog2/)
Continuing from Part 1, this article focuses on two critical backend security mechanisms: computing `SecretHash` using HMAC-SHA256 algorithms to prove backend authority to Cognito, highlighting the critical dangers of relying solely on `jwt.decode()`, and implementing cryptographic RSA JWT signature verification using `aws-jwt-verify` and JWKS public key sets.

### [Blog 3: Session Management with HttpOnly Cookies, Refresh Tokens, and RBAC Authorization with Cognito Groups](3.3-Blog3/)
This article analyzes storing JWT tokens inside HttpOnly Signed Cookies to mitigate XSS/CSRF exfiltration, token refresh workflows, and two-tier role-based access control combining NestJS RolesGuards with Cognito User Pool `ADMIN` Groups.