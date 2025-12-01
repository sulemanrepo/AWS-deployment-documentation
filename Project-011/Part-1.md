# Deploy Docker Containers on AWS ECS Fargate with Application Load Balancer

> This guide explains how to push Docker images to Amazon ECR Public, create Fargate task definitions, deploy containers on ECS, and expose them through an Application Load Balancer (ALB).

---

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Step 1: Push Docker Images to Amazon ECR Public](#step-1-push-docker-images-to-amazon-ecr-public)
- [Step 2: Create ECS Fargate Cluster & Task Definition](#step-2-create-ecs-fargate-cluster--task-definition)
- [Step 3: Create Application Load Balancer](#step-3-create-application-load-balancer)
- [Step 4: Create ECS Service](#step-4-create-ecs-service)
- [Step 5: Access Application](#step-5-access-application)
- [Conclusion](#conclusion)

---

## Overview

You will deploy two applications (**Node.js** and **PHP**) as containerized workloads in AWS ECS Fargate.  
The images will be stored in **Amazon ECR Public**, and the Node.js app will be exposed using an **Application Load Balancer (ALB)**.

Later, the PHP app can be deployed similarly by creating its own task definition and service.

---

## Architecture

```
                   AWS ECS Fargate Cluster
                      ┌─────────────────┐
                      │   Task: node-app│
                      │ Container: 3000 │
                      └───────┬────────┘
                              │ (IP Targets)
                  ┌───────────┴───────────┐
                  │ Application Load Balancer │
                  │     (Port 80 → 3000)     │
                  └───────────┬───────────┘
                              │
                            Users
```

---

# Step 1: Push Docker Images to Amazon ECR Public

### 1. Create public ECR repositories

Go to:

```
AWS Console → ECR → Public Repositories
```

Create:

- `node-app`
- `php-app`

---

## 🔹 Authenticate Docker to ECR Public

Replace the registry ID with your own ECR Public registry.

### Node App Login

```bash
aws ecr-public get-login-password --region us-east-1 | \
docker login --username AWS --password-stdin public.ecr.aws/p3d3v7x1
```

### Build, tag & push Node.js image

```bash
cd node-app
docker build -t node-app .
docker tag node-app:latest public.ecr.aws/p3d3v7x1/node-app:latest
docker push public.ecr.aws/p3d3v7x1/node-app:latest
```

---

### PHP App Login (same command)

```bash
aws ecr-public get-login-password --region us-east-1 | \
docker login --username AWS --password-stdin public.ecr.aws/p3d3v7x1
```

### Build, tag & push PHP image

```bash
cd php-app
docker build -t php-app .
docker tag php-app:latest public.ecr.aws/p3d3v7x1/php-app:latest
docker push public.ecr.aws/p3d3v7x1/php-app:latest
```

---

# Step 2: Create ECS Fargate Cluster & Task Definition

### 1. Create new ECS Cluster

Go to:  
`AWS Console → ECS → Create Cluster`

- Cluster name: `test-app`
- Template: **Networking Only (Fargate)**
- Create

---

## 2. Create a Task Definition (Node.js)

Go to:

`ECS → Task Definitions → Create New`

### Configure task:

| Setting | Value |
|--------|--------|
| Launch Type | Fargate |
| Task Role | None |
| Execution Role | Existing ecsTaskExecutionRole |
| Task Memory | 512 MiB or 1 GB |
| Task CPU | 0.25 vCPU |

---

## 3. Add Container (Node.js)

Add container:

- Name: **node-app**
- Image:  
  ```
  public.ecr.aws/p3d3v7x1/node-app:latest
  ```
- Port Mapping: **3000**
- Soft Memory Limit: **1GB**

Click **Create**.

---

# Step 3: Create Application Load Balancer (ALB)

Go to:

`EC2 → Load Balancers → Create Load Balancer`

### Configure ALB:

| Setting | Value |
|--------|--------|
| Name | app-balancer |
| Type | Application Load Balancer |
| Scheme | Internet-facing |
| Listener | HTTP : 80 |
| Availability Zones | Choose subnets in your VPC |

### Create Target Group:

- Type: **IP**
- Port: **3000**
- VPC: Same as ECS

Create target group.

---

# Step 4: Create ECS Service

Go to:

`ECS → Cluster → test-app → Create Service`

### Configure service:

- Launch Type: **Fargate**
- Task Definition: **node-app**
- Service Name: `node-app-service`
- Number of Tasks: **1**
- Cluster VPC: Select your VPC
- Subnets: Select public or private subnets

### Enable Load Balancing:

- Choose: **Application Load Balancer**
- Select: `app-balancer`
- Listener port: **80**
- Target Group: select the TG created earlier

Click **Create Service**.

Wait for it to reach:

```
ACTIVE ✓
RUNNING ✓
```

---

# Step 5: Access Application

Go to:

```
EC2 → Load Balancers → app-balancer
```

Copy the DNS name:

```
http://app-balancer-123456789.us-east-1.elb.amazonaws.com
```

Paste in browser → You should see your **Node.js app** running.

---

> 🔹 To deploy PHP app:  
Create another task definition + service in ECS, referencing the `php-app` ECR image.

---

# Conclusion

You have successfully:

✔ Built and pushed Docker images to Amazon ECR Public  
✔ Created an ECS Fargate cluster  
✔ Created task definitions for containerized workloads  
✔ Deployed apps using ECS Fargate services  
✔ Attached an Application Load Balancer for public access  

This setup is fully scalable, serverless, and production-ready.

---

**Author:** Your Name  
**Project:** ECS Fargate Deployment with ALB  
