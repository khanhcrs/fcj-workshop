---
title: "React 19 Frontend Integration, Business Discovery & Admin Dashboard"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

### 1. Overview

The **React 19 Frontend** application (Vite + TypeScript) provides a modern user experience connecting to **NestJS REST APIs**, managing session state via **Zustand**, attaching Bearer Tokens automatically via **Axios Interceptors**, and delivering dedicated workflows for **Businesses**, **Investors**, and **Administrators (Admin Dashboard)**.

---

### 2. Learning Objectives

- Understand client authentication state management using Zustand (`authStore.ts`).
- Configure **Axios Request Interceptors** to automatically attach **`Authorization: Bearer <accessToken>`** headers.
- Explore business discovery interfaces, smart filtering, and funding opportunity listings.
- Understand Admin Dashboard UI (`/admin/*`) operations, **RBAC** access control, and session authentication error troubleshooting.

---

### 3. State Management & Axios Interceptor (`authStore` & Axios)

Upon successful Cognito login, the Frontend stores the Access Token in `localStorage` and automatically attaches it to outgoing HTTP requests:

```typescript
// Axios Request Interceptor configuration in frontend/src/lib/api.ts
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

### 4. Investor Business Discovery Interface

Investors utilize web interfaces to search for companies and evaluate capital raising opportunities:
- Filter businesses by industry, location, employee size, and stage.
- Inspect financial highlights (`financialHighlights`) and historical funding rounds (`FundingRound`).
- Save companies (**`SavedBusiness`**), follow updates (**`BusinessFollow`**), and submit direct contact requests (**`ContactRequest`**).

![Business Listing](/images/workshop/frontend-business-listing.png)

*Figure 11. Business listing and discovery interface of Startups Blogs.*

---

### 5. Frontend Role-Based Access Control

When an authenticated user attempts to access administrative routes (`/admin/*`) without having Administrator privileges (not in the Cognito Group **`ADMIN`**), the frontend guards block navigation and display an **Access Denied** interface:

![Admin Access Denied](/images/workshop/admin-dashboard-dennied.png)

*Figure 12. Access Denied page displayed when a user does not have permission to access an administrative route.*

> **Note:** Frontend route protection improves navigation and user experience, but backend APIs must still enforce authorization through **`JwtAuthGuard`**, **`RolesGuard`**, and **Resource Ownership** checks.

---

### 6. Integration Troubleshooting: Admin Session Verification

During integration testing, if connection between the **Admin Dashboard** and **Amazon API Gateway** or **NestJS Backend** fails or session verification cannot complete (due to expired tokens or gateway routing issues), an error banner is presented:

![Admin API Session Failure](/images/workshop/admin-dashboard.png)

*Figure 13. Example of an Admin API/session verification failure during integration troubleshooting.*

---

### 7. Summary

We have successfully integrated the **React 19 Frontend** with the **NestJS Backend**, configured automated Bearer Token handling, and explored UI workflows for both Investors and Administrators. In the final hands-on module (5.7), we will automate the AWS cloud infrastructure using **Terraform** and execute resource cleanup.
