---
title: "Terraform Infrastructure Automation, Monitoring & Resource Cleanup"
date: 2024-01-01
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

### 1. Overview

The entire AWS Cloud Infrastructure for the **Startups Blogs** project is defined and automated 100% using **Terraform (Infrastructure as Code)** inside the `terraform/` directory.

This final hands-on module guides you through provisioning AWS infrastructure via Terraform, configuring monitoring with **Amazon CloudWatch**, and executing resource teardown (Resource Cleanup) to avoid incurring unexpected cloud costs.

---

### 2. Learning Objectives

- Master the cloud infrastructure deployment workflow using **Terraform** IaC code.
- Understand how to configure **Amazon CloudWatch Alarms** to monitor EC2 CPU Utilization metrics and dispatch email notifications via **Amazon SNS**.
- Execute clean cloud teardown using `terraform destroy` and clean local Docker environments using `docker compose down -v`.

---

### 3. Terraform Infrastructure Code Structure (`terraform/`)

Terraform configuration files inside `terraform/` manage 100% of cloud resources in region **`us-east-1`**:

- **`vpc.tf`**: Provisions the **Amazon VPC** (`10.0.0.0/16`), 2 Public Subnets, 2 Private Subnets, Internet Gateway, and Security Groups (`ec2_sg` **Port 3000**, `rds_sg` **Port 5432**).
- **`ec2.tf`**: Provisions the Ubuntu **Amazon EC2** backend compute instance running NestJS and assigns the **IAM Instance Profile** (`ec2_role`).
- **`rds.tf`**: Provisions **Amazon RDS PostgreSQL** (`db.t3.micro`) inside isolated Private Subnets.
- **`cognito.tf`**: Provisions **Amazon Cognito User Pool** and **Public App Client** (**`generate_secret = false`**).
- **`s3_cloudfront.tf`**: Provisions **Amazon S3 Frontend Bucket**, **Amazon S3 Media Bucket**, and **Amazon CloudFront** CDN Distribution.
- **`apigateway.tf`**: Provisions **Amazon API Gateway** (HTTP API) routing HTTPS proxy traffic to EC2.
- **`monitoring.tf`**: Provisions **Amazon CloudWatch Metric Alarm** (`CPUUtilization >= 70%`), CloudWatch Dashboard, and **Amazon SNS** Email notifications (`alert_email`).

---

### 4. Hands-on Implementation: Provisioning AWS Infrastructure with Terraform

#### Step 1: Open Terminal and navigate to the terraform directory

```bash
cd /Users/khanhtran/Project/Startup_Blogs/terraform
```

#### Step 2: Initialize Terraform providers and modules

```bash
terraform init
```

#### Step 3: Inspect the execution plan

```bash
terraform plan
```

#### Step 4: Provision AWS Cloud Infrastructure

```bash
terraform apply
```

Type `yes` when prompted in the Terminal to confirm resource creation on AWS (**`us-east-1`**).

---

### 5. System Monitoring with Amazon CloudWatch (`monitoring.tf`)

The `monitoring.tf` file deploys an automated observability solution:
- **CloudWatch Metric Alarm**: Automatically tracks `CPUUtilization` for the EC2 Backend instance.
- **Amazon SNS Email Notification**: Sends real-time email alerts to `var.alert_email` when CPU usage exceeds 70%.
- **CloudWatch Dashboard**: Provides a visual interface tracking performance metrics for both **Amazon EC2** and **Amazon RDS PostgreSQL**.

---

### 6. Hands-on Implementation: AWS Resource Teardown & Local Cleanup

> ⚠️ **CRITICAL WARNING**: Executing **`terraform destroy`** will permanently remove all provisioned AWS cloud resources. Ensure you have completed all testing before proceeding with destruction.

#### Step 1: Teardown all AWS Cloud Infrastructure via Terraform

```bash
cd /Users/khanhtran/Project/Startup_Blogs/terraform
terraform destroy
```

Type `yes` when prompted in the Terminal to release all cloud resources on AWS.

#### Step 2: Clean up local Docker container environments

```bash
cd /Users/khanhtran/Project/Startup_Blogs
docker compose down -v
```

The `docker compose down -v` command stops `startups_blogs_db` and `startups_blogs_minio` containers and removes temporary Docker volumes.

---

### 7. Workshop Summary & Conclusion

Congratulations on completing all **7 hands-on modules** of the Workshop! Throughout this course, you have:

1. Understood the multi-tier Enterprise web application architecture combining **React 19**, **NestJS**, **PostgreSQL**, and **Amazon Cognito**.
2. Automated 100% of AWS cloud infrastructure (**Amazon VPC**, **Amazon EC2**, **Amazon RDS**, **Amazon Cognito**, **Amazon API Gateway**, **Amazon S3**, **Amazon CloudFront**, **Amazon CloudWatch**, **Amazon SNS**) using **Terraform (IaC)**.
3. Mastered configuring **Amazon Cognito Public App Client** mode and Cognito Group **`ADMIN`** synchronization.
4. Implemented REST APIs for business profiles, funding opportunities, S3 image uploads, and cryptographic RSA JWT verification via **`aws-jwt-verify`**.
5. Conducted security reviews for dual-layer authorization (**`JwtAuthGuard`**, **`RolesGuard`**, Resource Ownership) and mastered resource teardown procedures.
