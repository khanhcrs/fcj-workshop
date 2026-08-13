---
title: "Amazon Cognito User Pool, Public App Client & RBAC Configuration"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

### 1. Overview

**Amazon Cognito User Pool** provides a cloud-native identity management and user authentication solution for the **Startups Blogs** application in region **`us-east-1`**.

This module guides you through creating a **Cognito User Pool**, configuring a **Public App Client** (**`generate_secret = false`**), enabling **`USER_PASSWORD_AUTH`** flow, and creating the Cognito User Group **`ADMIN`** for role authorization.

> ⚠️ **Key Architecture Note**: The application utilizes a **Public App Client** (no Client Secret Key generated) matching modern SPA web patterns. Therefore, the system does **not** use `SecretHash` or `COGNITO_CLIENT_SECRET`.

---

### 2. Learning Objectives

- Create an **Amazon Cognito User Pool** supporting Email sign-in and 6-digit OTP verification.
- Configure an App Client as a **Public App Client** (**`generate_secret = false`**).
- Create the Cognito User Group **`ADMIN`** for Administrator role access control.
- Validate the **JWKS** endpoint URL (`.well-known/jwks.json`) in region **`us-east-1`**.

---

### 3. Hands-on Step-by-Step Implementation

#### Step 1: Create Cognito User Pool
1. Log in to the **AWS Management Console** in region **`us-east-1`** (N. Virginia) (or inspect automation in `terraform/cognito.tf`).
2. Open **Amazon Cognito** service -> Click **Create user pool**.
3. Under **Authentication providers**: Select **Cognito user pool**.
4. Under **Sign-in options**: Select **Email**.
5. Under **Password policy**: Choose minimum 8 characters with uppercase, lowercase, and numbers.

![Cognito User Pool Overview](/images/workshop/cognito-user-pool-overview.png)

*Figure 7. Amazon Cognito User Pool overview in the us-east-1 region.*

#### Step 2: Create Public App Client (No Client Secret)
1. Under **App clients**: Name the client `startups-blogs-app`.
2. **Client type**: Select **Public client**.
3. **Client secret**: Ensure **Don't generate a client secret** is selected (**`generate_secret = false`**).
4. **Authentication flows**: Enable **`ALLOW_USER_PASSWORD_AUTH`** and **`ALLOW_REFRESH_TOKEN_AUTH`**.

![Cognito Public App Client](/images/workshop/cognito-app-client.png)

*Figure 8. Cognito Public App Client configuration for Startups Blogs.*

#### Step 3: Create Cognito User Group ADMIN
1. Navigate to **User management** -> Select **Groups** -> Click **Create group**.
2. Set Group Name: **`ADMIN`**.
3. Users belonging to **`ADMIN`** will have the claim `cognito:groups: ["ADMIN"]` encoded in their **JWT** access tokens.

![Cognito ADMIN Group](/images/workshop/cognito-admin-group.png)

*Figure 9. ADMIN group configuration in Amazon Cognito User Pool.*

#### Step 4: Inspect Authenticated User List

<!-- ================================================== -->
<!-- IMAGE PLACEHOLDER: COGNITO_USERS_SANITIZED -->
<!-- ================================================== -->

> **Figure 10. Authenticated users in the Cognito User Pool.**  
> *The screenshot will be added after personal email information is sanitized.*

<!-- ================================================== -->

---

### 4. Environment Parameters

Record these values to populate in your NestJS backend `.env` configuration file:

```env
AWS_REGION=us-east-1
COGNITO_USER_POOL_ID=us-east-1_xxxxxxxxx
COGNITO_CLIENT_ID=xxxxxxxxxxxxxxxxxxxxxxxxxx
```

---

### 5. Summary

We have created an **Amazon Cognito User Pool** in **Public App Client** mode and configured the **`ADMIN`** group. In the next module (5.5), we will integrate this identity service into the **NestJS REST API** using **`aws-jwt-verify`**.
