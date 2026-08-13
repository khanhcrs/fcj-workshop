---
title: "NestJS REST API Integration, S3 Uploads & RSA JWT Verification"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

### 1. Overview

The **NestJS Backend** server processes **REST API** requests, uploads media files to **Amazon S3**, cryptographically verifies **RSA JWT** digital signatures using **`aws-jwt-verify`**, and enforces multi-layer authorization controls.

---

### 2. Learning Objectives

- Understand the structure of Controllers and Services within NestJS modular architecture.
- Implement cryptographic **RSA JWT** verification using **`aws-jwt-verify`** against **Cognito JWKS** in **`us-east-1`**.
- Configure the media upload endpoint `POST /upload` using **`@aws-sdk/client-s3`**.
- Utilize **Swagger UI** for testing and documenting **REST API** endpoints.

---

### 3. NestJS Backend REST API Architecture

NestJS codebase is organized into primary functional modules:

- **Businesses (`BusinessesController`)**:
  - `GET /businesses`: Public business discovery listing (`isVerified: true`, `status: APPROVED`).
  - `POST /businesses`: Create new business profile (Requires **`JwtAuthGuard`**).
  - `PUT /businesses/:id`: Update business profile (Validates `business.ownerId === user.userId`).
  - `PUT /businesses/admin/:id/status`: Admin approval workflow (**`PENDING`** → **`APPROVED`**).
- **Funding Opportunities (`FundingOpportunitiesController`)**:
  - `POST /businesses/:businessId/funding-opportunities`: Publish capital raising listing.
  - `GET /businesses/:businessId/funding-opportunities`: Retrieve funding opportunities.
- **Upload (`UploadController`)**:
  - `POST /upload`: Uploads image files (logos, covers, avatars) to **Amazon S3 Media Bucket** / **MinIO** via **`@aws-sdk/client-s3`**.
- **Change Proposals (`ProposalsController`)**:
  - `POST /proposals`: Create JSON diff change proposal.
  - `POST /proposals/:id/decision`: Admin decision (**`APPROVED`**) applies JSON diff to **PostgreSQL** inside `prisma.$transaction`.

---

### 4. Cryptographic RSA JWT Verification via `aws-jwt-verify`

On the Backend, **`JwtAuthGuard`** extracts tokens from the **`Authorization: Bearer <accessToken>`** HTTP header and validates RSA digital signatures using **`aws-jwt-verify`**:

```typescript
// Excerpt from backend/src/auth/guards/jwt-auth.guard.ts
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

    // Cryptographically verify RSA signature against Cognito JWKS endpoint
    const payload = await this.verifier.verify(token);
    
    // Automatically provision user account in PostgreSQL
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

### 5. Swagger REST API Documentation Interface

NestJS integrates `@nestjs/swagger` to automatically generate documentation and test REST API endpoints:

![Swagger API Documentation](/images/workshop/swagger-api.png)

*Figure 10. Swagger interface for testing and documenting the NestJS REST API.*

---

### 6. Common HTTP Error Troubleshooting

- **HTTP 401 Unauthorized**:
  - Missing **`Authorization: Bearer <token>`** header.
  - Token expired or mismatched Cognito User Pool ID / Client ID.
- **HTTP 403 Forbidden**:
  - User lacks **`ADMIN`** role when accessing administrative routes (**`RolesGuard`**).
  - User does not match resource owner (`business.ownerId !== userId`).

---

### 7. Summary

We have successfully integrated the **NestJS REST API** with **`aws-jwt-verify`** to cryptographically validate **RSA** signatures from **Amazon Cognito** and configured S3 uploads. In the next module (5.6), we will integrate the **React 19 Frontend** and **Admin Dashboard**.
