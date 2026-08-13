---
title: "Local Development Environment, Docker & PostgreSQL Setup"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

### 1. Overview

Before configuring **Amazon Cognito** cloud identity services and automating AWS infrastructure using **Terraform**, we must prepare the local development environment.

Containerizing **PostgreSQL** and **MinIO** object storage using **Docker Compose** ensures a consistent offline development workflow before pushing code to the AWS Cloud.

---

### 2. Learning Objectives

- Install and prepare required software development tools (**Node.js**, **Docker**, **Terraform**, **AWS CLI**).
- Launch local **PostgreSQL 15** and **MinIO** S3 container instances via **Docker Compose**.
- Verify active container listening ports (**Port 5433** for DB, **Ports 9000/9001** for MinIO S3).

---

### 3. Prerequisites & Tools

- **Node.js**: Version Node.js 18+ or 20+ LTS.
- **Docker & Docker Desktop**: Containerization engine.
- **Terraform**: Version v1.5+ for automated cloud infrastructure provisioning.
- **AWS CLI**: Installed and configured with default region **`us-east-1`** (N. Virginia).

---

### 4. Local Container Architecture

The `docker-compose.yml` file in the project defines two core containerized services:

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    container_name: startups_blogs_db
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgrespassword
      POSTGRES_DB: startups_blogs
    ports:
      - '5433:5432' # Host port 5433 mapped to container port 5432

  minio:
    image: minio/minio:RELEASE.2023-09-04T19-57-37Z
    container_name: startups_blogs_minio
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadminpassword
    ports:
      - '9000:9000' # S3 API Endpoint
      - '9001:9001' # Web Console Interface
    command: server /data --console-address ":9001"
```

- **PostgreSQL 15 (`startups_blogs_db`)**: Local relational database listening on host **Port 5433**.
- **MinIO (`startups_blogs_minio`)**: S3-compatible local object storage listening on host **Port 9000** (API) and **Port 9001** (Console).

---

### 5. Hands-on Step-by-Step Implementation

#### Step 1: Open Terminal and navigate to the project directory

```bash
cd /Users/khanhtran/Project/Startup_Blogs
```

#### Step 2: Launch Docker container services in background mode

Run `docker compose up -d` to launch containers in detached mode.

```bash
docker compose up -d
```

#### Step 3: Verify container running status

```bash
docker ps
```

---

### 6. Expected Result

The containers `startups_blogs_db` and `startups_blogs_minio` should be in the **Running** state (Healthy):

![Docker Compose](/images/workshop/docker-compose.png)

*Figure 3. Starting PostgreSQL and MinIO using Docker Compose.*

---

### 7. Troubleshooting Tips

- **Port 5433 conflict**: Check if another PostgreSQL service is occupying the port using `lsof -i :5433`.
- **Docker daemon inactive**: Ensure **Docker Desktop** is launched before executing `docker compose`.

---

### 8. Summary

We have successfully launched local **PostgreSQL** and **MinIO** container environments via **Docker Compose**. In the next module (5.3), we will initialize the database schema using **Prisma ORM** and seed test data.