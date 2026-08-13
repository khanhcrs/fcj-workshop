---
title: "AWS Infrastructure Setup: VPC, Amazon RDS PostgreSQL & EC2 Backend Deployment"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

### Overview

Building a secure and scalable cloud infrastructure is a core prerequisite for the **Startups Blogs** enterprise web application. This module provides a hands-on step-by-step guide to provisioning an **Amazon VPC** isolated cloud network, creating **Subnets**, configuring an **Internet Gateway**, setting up **Security Groups**, deploying an **Amazon RDS PostgreSQL** database instance, and hosting the **NestJS Backend** 24/7 on an **Amazon EC2** compute instance.

---

## 5.3.1 VPC, Subnets, Internet Gateway & Security Groups

#### 1. Overview
The networking layer establishes the primary security perimeter for cloud resources. An **Amazon VPC** isolates compute servers and database instances from unauthorized public Internet access.

#### 2. Learning Objectives
- Provision an Amazon VPC named **`Startups-Blogs-vpc`** with CIDR block **`10.0.0.0/16`**.
- Configure network subnets: **`10.0.0.0/20`**, **`10.0.128.0/20`**, and **`10.0.200.0/24`**.
- Create an **Internet Gateway** named **`Startups-Blogs-igw`** to route outbound internet traffic from Public Subnets.
- Configure firewall security rules for the **EC2 Security Group** and **RDS Security Group**.

#### 3. Hands-on Step-by-Step Implementation

##### Step 1: Create Amazon VPC
Navigate to AWS Console -> Open **VPC** service -> Set VPC Name: **`Startups-Blogs-vpc`** and enter IPv4 CIDR: **`10.0.0.0/16`**.

![Create VPC](/images/workshop/5.3/5.3.1-step1-create-vpc.png)

*Figure 1. Amazon VPC creation interface for Startups Blogs.*

##### Step 2: Configure Network Subnets
Create subnets within the VPC using the specified CIDR ranges:
- Public Subnet: **`10.0.0.0/20`**
- Private Subnet 1: **`10.0.128.0/20`**
- Private Subnet 2: **`10.0.200.0/24`**

![Subnets](/images/workshop/5.3/5.3.1-step2-subnets.png)

*Figure 2. Subnets list inside Amazon VPC.*

##### Step 3: Create and Attach Internet Gateway
Create an Internet Gateway named **`Startups-Blogs-igw`** and execute **Attach to VPC** targeting `Startups-Blogs-vpc`.

![Internet Gateway](/images/workshop/5.3/5.3.1-step3-internet-gateway.png)

*Figure 3. Internet Gateway configuration for outbound internet access.*

##### Step 4: Configure EC2 Security Group
Create a Security Group for the EC2 Backend server and define **Inbound Rules**:
- **SSH (Port 22)**: Allow developer SSH access.
- **Custom TCP (Port 3000)**: Allow API Gateway/Frontend access to the NestJS application.

![EC2 Security Group](/images/workshop/5.3/5.3.1-step4-ec2-security-group.png)

*Figure 4. Inbound Rules configuration for EC2 Security Group.*

##### Step 5: Configure RDS Security Group
Create a Security Group for PostgreSQL database, defining the **Inbound Rule**:
- **PostgreSQL (Port 5432)**: **Restrict access exclusively** to the **EC2 Security Group** (internal communication only, no public internet exposure).

![RDS Security Group](/images/workshop/5.3/5.3.1-step5-rds-security-group.png)

*Figure 5. Inbound Rules configuration for RDS Security Group restricting access to EC2.*

#### 4. Expected Result
The isolated **`Startups-Blogs-vpc`** network is active with gateway **`Startups-Blogs-igw`** and two custom Security Groups attached.

#### 5. Troubleshooting Tips
- **SSH connection to EC2 fails**: Verify the EC2 Security Group allows Port **`22`** inbound traffic and the attached **Route Table** routes `0.0.0.0/0` traffic to the Internet Gateway.

---

## 5.3.2 Amazon RDS PostgreSQL Deployment

#### 1. Overview
**Amazon RDS PostgreSQL** provides a managed relational database service with automated backups and security controls for Startups Blogs.

#### 2. Learning Objectives
- Provision an RDS Database Instance named **`startups-blogs-db`** running the **PostgreSQL** engine on instance type **`db.t4g.micro`**.
- Configure master credentials and disable **Public Access** for data protection.
- Retrieve the **RDS Endpoint** and assemble the `DATABASE_URL` connection string.

#### 3. Hands-on Step-by-Step Implementation

##### Step 1: Select Database Engine
Navigate to Amazon RDS -> Click **Create database** -> Select **PostgreSQL** engine (**Standard Create** mode).

![Select Engine](/images/workshop/5.3/5.3.2-step1-database-engine.png)

*Figure 6. Database Engine selection step for Amazon RDS.*

> **Note:** The example creation screen illustrates Aurora PostgreSQL selection; for the actual Startups Blogs deployment, select the standard **PostgreSQL** engine on instance type **`db.t4g.micro`** with DB Identifier: **`startups-blogs-db`**.

##### Step 2: Configure Database Credentials
Enter **Master Username** and set a secure **Master Password** for the database instance.

![Database Settings](/images/workshop/5.3/5.3.2-step2-database-settings.png)

*Figure 7. Master Username and Password setup for RDS.*

##### Step 3: Network & Security Connectivity
Select VPC deployment options:
- **VPC**: Select **`Startups-Blogs-vpc`**.
- **Publicly accessible**: Select **No** (disable public access).
- **VPC Security Group**: Select the RDS Security Group created in Section 5.3.1.

![Connectivity Settings](/images/workshop/5.3/5.3.2-step3-database-connectivity.png)

*Figure 8. Network VPC and security connectivity settings for RDS Instance.*

> **Note:** The initial connectivity setup screen illustrates Default VPC selection. For the final Startups Blogs deployment, select **`Startups-Blogs-vpc`** and attach the custom RDS Security Group configured in 5.3.1.

##### Step 4: Verify Database Instance Availability
Wait for database creation to complete until the **Status** column displays **Available**.

![RDS Available](/images/workshop/5.3/5.3.2-step4-rds-available.png)

*Figure 9. Amazon RDS Instance in Available status.*

##### Step 5: Retrieve RDS Endpoint
Open Database details -> Record the **Endpoint** address and **Port** (5432).

![RDS Endpoint](/images/workshop/5.3/5.3.2-step5-rds-endpoint.png)

*Figure 10. Endpoint and Port information for Amazon RDS PostgreSQL.*

Configure the connection string in the NestJS Backend `.env` file:
```env
DATABASE_URL="postgresql://<username>:<password>@<rds-endpoint>:5432/startups_blogs?schema=public"
```

#### 4. Expected Result
The **`startups-blogs-db`** instance reaches **Available** status and is ready for secure connections from the EC2 Backend.

#### 5. Troubleshooting Tips
- **Backend connection timeout**: Verify that the RDS Security Group allows Inbound Port **`5432`** traffic from the EC2 Security Group.

---

## 5.3.3 Amazon EC2 & NestJS Backend Deployment

#### 1. Overview
The **Amazon EC2** compute instance serves as the execution environment for the **NestJS Backend** application, connecting to **Amazon RDS PostgreSQL** and **Amazon Cognito**.

#### 2. Learning Objectives
- Provision an EC2 compute instance named **`startups-blogs-backend`** running **Ubuntu Server**.
- Configure SSH Key Pair **`startups-key`** for remote server administration.
- Verify environment runtime **Node.js v20.20.2**, **npm 10.8.2**, and install **PM2 7.0.3**.
- Configure safe `.env` environment variables and run NestJS via PM2 process manager.

#### 3. Hands-on Step-by-Step Implementation

##### Step 1: Provision Amazon EC2 Instance
Navigate to EC2 -> Click **Launch Instance** -> Set Name: **`startups-blogs-backend`** -> Select OS: **Ubuntu Server**.

![Launch EC2](/images/workshop/5.3/5.3.3-step1-launch-ec2.png)

*Figure 11. Amazon EC2 instance name and OS configuration step.*

> **Note:** The launch selection screen shows `t3.micro` instance selection. During test execution, `t2.micro` or `t3.micro` may be selected based on AWS account tier availability.

##### Step 2: Configure SSH Key Pair
Create or attach SSH Key Pair **`startups-key`** for secure remote connection.

![Key Pair](/images/workshop/5.3/5.3.3-step2-key-pair.png)

*Figure 12. Selecting SSH Key Pair for EC2 Instance.*

##### Step 3: Verify EC2 Instance State
After launching, monitor the EC2 dashboard until **Instance state** displays **Running**.

![EC2 Running](/images/workshop/5.3/5.3.3-step3-ec2-running.png)

*Figure 13. EC2 Instance in Running state.*

##### Step 4: Verify Node.js & PM2 Environment
Connect via SSH and verify installed environment runtimes:
```bash
node -v
npm -v
pm2 -v
```

![Node.js Environment](/images/workshop/5.3/5.3.3-step4-nodejs-pm2.png)

*Figure 14. Verifying Node.js v20.20.2, npm 10.8.2, and PM2 7.0.3 on EC2.*

##### Step 5: Configure Environment Variables (`.env`)

<!-- ================================================== -->
<!-- IMAGE PLACEHOLDER: BACKEND_ENV_SANITIZED -->
<!-- ================================================== -->

> **Figure 15. Sanitized example configuration for Backend environment variables.**

Safe configuration template excluding real secrets:
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

> ⚠️ **Security Warning**: `AWS_S3_ACCESS_KEY` and `AWS_S3_SECRET_KEY` are sensitive confidential credentials. Never commit `.env` files containing real production keys to public Git repositories.

##### Step 6: Process Management with PM2 & API Verification
Build the application and start the process using **PM2**:
```bash
npm run build
pm2 start dist/main.js --name "startups-blogs-backend"
pm2 status
curl -I http://localhost:3000
```

![PM2 Running](/images/workshop/5.3/5.3.3-step6-pm2-backend-running.png)

*Figure 16. Verifying online status of NestJS Backend process managed by PM2.*

> 💡 **Production CORS Note**: In production environments, NestJS CORS configuration should not maintain open `Access-Control-Allow-Origin: *` settings but rather restrict access to trusted Frontend/CloudFront domains.

#### 4. Expected Result
The command `pm2 status` shows application `startups-blogs-backend` in **online** status and `curl -I http://localhost:3000` returns **`HTTP/1.1 200 OK`**.

#### 5. Troubleshooting Tips
- **PM2 status displays `errored`**: Inspect logs via `pm2 logs` (common causes include invalid `DATABASE_URL` credentials or missing Cognito environment variables).

---

## 5.3.4 Amazon S3 & CloudFront Frontend Delivery

#### 1. Overview
**Amazon S3** object storage and the **Amazon CloudFront** content delivery network (CDN) host the static **React 19 SPA** web build files and deliver media assets (logos, avatars, covers) for the **Startups Blogs** platform.

#### 2. Learning Objectives
- Distinguish between project S3 Buckets (**`startups-blogs-frontend`** and **`startups-blogs-media`**).
- Configure **CORS** rules to allow secure web application interaction with S3.
- Provision an **Amazon CloudFront Distribution** as the CDN delivery layer.
- Configure **Custom Error Pages** (403 & 404 → `/index.html`) to support client-side **React Router SPA** navigation.

#### 3. Hands-on Step-by-Step Implementation

##### Step 1: Initialize Amazon S3 Buckets
The **Startups Blogs** project utilizes primary S3 Buckets:
- **`startups-blogs-frontend`**: Hosts static web build assets (HTML, JS, CSS) for the **React 19** application.
- **`startups-blogs-media`**: Stores uploaded business logos, avatars, and attachments (`POST /upload`).

![S3 Media Bucket](/images/workshop/5.3/5.3.4-step1-media-bucket.png)

*Figure 17. Amazon S3 Media Bucket configuration interface.*

> ⚠️ **Security Note**: Initial testing bucket creation screens may display Block Public Access disabled. In production architectures, S3 Buckets should remain private, allowing origin access strictly via Amazon CloudFront Origin Access Control (OAC).

##### Step 2: Configure S3 Cross-Origin Resource Sharing (CORS)
Define JSON CORS rules enabling HTTP web request interactions:

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

*Figure 18. S3 CORS rules configuration allowing HTTP API interactions.*

> 💡 **Deployment Note**: Wildcard CORS configuration (`AllowedOrigins: ["*"]`) is suitable for workshop testing environments. In production, `AllowedOrigins` should be restricted exclusively to the application domain.

##### Step 3: List Amazon S3 Buckets
Verify the list of created S3 buckets on the Amazon S3 Console.

![S3 Buckets List](/images/workshop/5.3/5.3.4-step3-s3-buckets.png)

*Figure 19. List of Amazon S3 Buckets supporting Startups Blogs.*

##### Step 4: Configure Amazon CloudFront Distribution
Provision a CloudFront Distribution targeting the **`startups-blogs-frontend`** S3 Bucket origin:
- **Status**: **Enabled**
- **Type**: **Standard**
- **Origin**: `startups-blogs-frontend.s3.us-east-1.amazonaws.com`

![CloudFront Distribution](/images/workshop/5.3/5.3.4-step5-distribution.png)

*Figure 20. Provisioning and activating the Amazon CloudFront CDN Distribution.*

##### Step 5: Configure SPA Routing (Custom Error Pages)
The **React SPA** handles client-side routing via **React Router**. When users navigate directly to routes such as `/businesses`, `/profile`, or `/admin`, S3 returns HTTP 403 or 404 errors.

Configure **Custom Error Responses** on CloudFront:
- **HTTP Error Code**: **`403`** & **`404`**
- **Response Page Path**: **`/index.html`**
- **HTTP Response Code**: **`200`**

![CloudFront SPA Error Pages](/images/workshop/5.3/5.3.4-step6-spa-error-pages.png)

*Figure 21. CloudFront Custom Error Pages configuration for React Router SPA navigation.*

#### 4. Expected Result
Users accessing the CloudFront domain load the React SPA seamlessly and navigate client-side routes without encountering 404 errors.

#### 5. Troubleshooting Tips
- **404 Error upon page refresh**: Verify that Custom Error Responses (403/404 → `/index.html` HTTP 200) are properly configured on the CloudFront Distribution.

---

## 5.3.5 Amazon API Gateway Integration

#### 1. Overview
**Amazon API Gateway** acts as the public API Entry Point between the **React 19 Frontend** and the **NestJS Backend** running on EC2.

#### 2. Learning Objectives
- Provision an **HTTP API** named **`startups-blogs-api`** in region **`us-east-1`**.
- Create the proxy route **`ANY /{proxy+}`** connecting to the EC2 Backend on Port 3000.
- Configure API Gateway **CORS** rules compatible with **Bearer Token** authentication.
- Retrieve the **Invoke URL** to populate the `VITE_API_URL` environment variable.

#### 3. Hands-on Step-by-Step Implementation

##### Step 1: Create Amazon HTTP API
Navigate to API Gateway -> Click **Build HTTP API** -> API Name: **`startups-blogs-api`** -> Protocol: **HTTP** -> Endpoint Type: **Regional**.

![Create API Gateway](/images/workshop/5.3/5.3.5-step1-create-api.png)

*Figure 22. Creating Amazon API Gateway (HTTP API) for Startups Blogs.*

##### Step 2: Configure Backend Integration Proxy
Create an **HTTP URI** integration pointing to the NestJS Backend on EC2:
- **Target**: `http://<EC2_PUBLIC_IP>:3000/{proxy}`
- **Timeout**: **`30000`** milliseconds

![Backend Integration](/images/workshop/5.3/5.3.5-step2-integration.png)

*Figure 23. HTTP Proxy Integration forwarding requests to the EC2 Backend.*

> 💡 **Architecture Note**: Direct HTTP integration to an EC2 public IP is suitable for workshop testing. Production architectures should utilize an internal **VPC Link** connecting to a private load balancer.

##### Step 3: Configure Route `ANY /{proxy+}`
Define the proxy route **`ANY /{proxy+}`** to forward all HTTP methods (`GET`, `POST`, `PUT`, `DELETE`) and paths to NestJS.

![API Route](/images/workshop/5.3/5.3.5-step3-route.png)

*Figure 24. Proxy route ANY /{proxy+} configuration on API Gateway.*

> 🔐 **Security & Authentication Note**: The interface displays *"No authorizer attached to this route"*. This does **not** indicate missing application authentication; rather, JWT verification is delegated to the NestJS Backend via **`JwtAuthGuard`**, **`aws-jwt-verify`**, and **Cognito JWKS**. API Gateway functions primarily as the routing proxy layer.

##### Step 4: Configure API Gateway CORS
Configure API Gateway CORS matching the **Bearer Token** architecture:
- **AllowedOrigins**: `*`
- **AllowedHeaders**: `*`
- **AllowedMethods**: `GET`, `POST`, `OPTIONS`, `PATCH`, `PUT`, `DELETE`
- **AllowCredentials**: **NO** (matches the `Authorization: Bearer <accessToken>` header pattern).

![API Gateway CORS](/images/workshop/5.3/5.3.5-step4-cors.png)

*Figure 25. API Gateway CORS rules configuration.*

##### Step 5: Retrieve Invoke URL
Select Stage **`$default`** (enable **Auto-deploy**) -> Retrieve the **Invoke URL**.

![API Gateway Invoke URL](/images/workshop/5.3/5.3.5-step5-invoke-url.png)

*Figure 26. Official Amazon API Gateway Invoke URL.*

Configure the Invoke URL in the React Frontend environment file:
```env
VITE_API_URL=https://<api-id>.execute-api.us-east-1.amazonaws.com
```

#### 4. Expected Result
All HTTP API requests sent to the API Gateway Invoke URL are forwarded to the NestJS Backend on EC2 and return responses successfully.

#### 5. Troubleshooting Tips
- **500 Internal Server Error / 503 Service Unavailable**: Verify that the EC2 Security Group allows Port **`3000`** inbound traffic from API Gateway.

---

### Complete Infrastructure Architecture Model

Upon completing Sections 5.3.1 through 5.3.5, the complete AWS cloud infrastructure for **Startups Blogs** operates as follows:

```text
User Browser (React 19 SPA)
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

Supporting Services:
- Amazon Cognito User Pool (Identity authentication & JWKS us-east-1)
- Amazon S3 Media Bucket (Image & attachment storage)
```

---

### Summary

We have successfully built and connected the complete AWS Cloud infrastructure for **Startups Blogs**: initializing **Amazon VPC**, **Amazon RDS PostgreSQL**, **Amazon EC2**, **Amazon S3**, **Amazon CloudFront**, and **Amazon API Gateway**. In the next module (5.4), we will dive into configuring **Amazon Cognito User Pool** and RBAC authorization.
