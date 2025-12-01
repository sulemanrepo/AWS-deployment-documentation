# GitHub Actions CI/CD Pipeline for AWS ECS Fargate Deployment

> This documentation provides a complete, production-ready guide for automating Docker image building, pushing to Amazon ECR, updating ECS task definitions, and deploying to ECS Fargate using GitHub Actions.  
> It includes all steps, commands, IAM permissions, secrets configuration, and full pipeline YAML.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Step 1: Create ECR Repository](#step-1-create-ecr-repository)
- [Step 2: Create IAM User for GitHub Actions](#step-2-create-iam-user-for-github-actions)
- [Step 3: Add GitHub Repository Secrets](#step-3-add-github-repository-secrets)
- [Step 4: Prepare ECS Cluster & Task Definition](#step-4-prepare-ecs-cluster--task-definition)
- [Step 5: Create GitHub Actions Pipeline](#step-5-create-github-actions-pipeline)
- [Step 6: Push Code → Auto-Deploy](#step-6-push-code--auto-deploy)
- [Step 7: Validate Deployment](#step-7-validate-deployment)
- [Troubleshooting](#troubleshooting)
- [Conclusion](#conclusion)

---

## Overview

This workflow automates:

1. Detect new code pushes on GitHub  
2. Build Docker image from Dockerfile  
3. Authenticate to Amazon ECR  
4. Push image to ECR  
5. Update ECS Task Definition with new image  
6. Redeploy ECS Service on Fargate  
7. Automatically roll out new version  

This is a **zero-downtime CI/CD deployment** for containerized applications.

---

## Architecture

```
GitHub Push → GitHub Actions → Build Docker Image → Push to ECR →  
Update ECS Task Definition → Deploy to ECS Fargate → Serve via Load Balancer
```

---

## Prerequisites

Before starting, make sure you have:

- An ECS Cluster (Fargate)
- A Task Definition created
- A running ECS Service using that task definition
- An ECR repository to store container images
- GitHub repository containing your application & Dockerfile

---

# Step 1: Create ECR Repository

Go to:

```
AWS Console → ECR → Create Repository
```

Example:

- Repository name: `node-app`  
- Type: **Private** (recommended)

Copy the repository URI:

```
123456789012.dkr.ecr.us-east-1.amazonaws.com/node-app
```

---

# Step 2: Create IAM User for GitHub Actions

### Create a new IAM user:

```
IAM → Users → Create User → Programmatic Access
```

Attach policy:

```
AmazonEC2ContainerRegistryFullAccess
AmazonECS_FullAccess
```

Or create a least-privileged policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    { "Effect": "Allow", "Action": "ecr:*", "Resource": "*" },
    { "Effect": "Allow", "Action": "ecs:*", "Resource": "*" },
    { "Effect": "Allow", "Action": "iam:PassRole", "Resource": "*" }
  ]
}
```

Save the **Access Key ID** and **Secret Access Key**.

---

# Step 3: Add GitHub Repository Secrets

Open GitHub → your repo →  
**Settings → Secrets and Variables → Actions**

Add these:

| Secret Name | Value |
|-------------|-------|
| `AWS_ACCESS_KEY_ID` | IAM User Access Key |
| `AWS_SECRET_ACCESS_KEY` | IAM Secret Key |
| `AWS_REGION` | e.g. `us-east-1` |
| `ECR_REPOSITORY` | `node-app` |
| `ECR_REGISTRY` | `123456789012.dkr.ecr.us-east-1.amazonaws.com` |
| `ECS_CLUSTER` | Name of ECS cluster |
| `ECS_SERVICE` | Name of service |
| `ECS_TASK_DEFINITION` | Path to taskdef.json |

---

# Step 4: Prepare ECS Cluster & Task Definition

### 1. ECS Cluster
Create:

```
ECS → Create Cluster → Fargate → Cluster Name: my-app-cluster
```

### 2. Task Definition JSON (required for pipeline)

Create a file in your repo:

```
taskdef.json
```

Example:

```json
{
  "family": "node-app-task",
  "networkMode": "awsvpc",
  "executionRoleArn": "arn:aws:iam::123456789012:role/ecsTaskExecutionRole",
  "containerDefinitions": [
    {
      "name": "node-app",
      "image": "<IMAGE_URI>",
      "cpu": 256,
      "memory": 512,
      "portMappings": [
        {
          "containerPort": 3000,
          "protocol": "tcp"
        }
      ],
      "essential": true
    }
  ],
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512"
}
```

`<IMAGE_URI>` will be replaced automatically by GitHub Actions.

---

# Step 5: Create GitHub Actions Pipeline

Create folder:

```
.github/workflows/deploy.yml
```

---

## Full CI/CD Pipeline (ready to run)

```yaml
name: Deploy to AWS ECS Fargate

on:
  push:
    branches: ["main"]

jobs:
  deploy:
    name: Deploy Container to ECS Fargate
    runs-on: ubuntu-latest

    steps:
    - name: Checkout source
      uses: actions/checkout@v3

    - name: Configure AWS Credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ secrets.AWS_REGION }}

    - name: Log in to Amazon ECR
      uses: aws-actions/amazon-ecr-login@v1

    - name: Build, Tag, and Push Image to ECR
      run: |
        IMAGE_TAG=${{ github.sha }}
        docker build -t ${{ secrets.ECR_REGISTRY }}/${{ secrets.ECR_REPOSITORY }}:$IMAGE_TAG .
        docker push ${{ secrets.ECR_REGISTRY }}/${{ secrets.ECR_REPOSITORY }}:$IMAGE_TAG
        echo "IMAGE_TAG=$IMAGE_TAG" >> $GITHUB_ENV

    - name: Render Amazon ECS task definition
      id: taskdef
      uses: aws-actions/amazon-ecs-render-task-definition@v1
      with:
        task-definition: taskdef.json
        container-name: node-app
        image: ${{ secrets.ECR_REGISTRY }}/${{ secrets.ECR_REPOSITORY }}:${{ env.IMAGE_TAG }}

    - name: Deploy to Amazon ECS
      uses: aws-actions/amazon-ecs-deploy-task-definition@v1
      with:
        task-definition: ${{ steps.taskdef.outputs.task-definition }}
        service: ${{ secrets.ECS_SERVICE }}
        cluster: ${{ secrets.ECS_CLUSTER }}
        wait-for-service-stability: true
```

---

# Step 6: Push Code → Auto-Deploy

Once everything is configured:

```bash
git add .
git commit -m "Enable ECS deployment pipeline"
git push origin main
```

GitHub Actions will automatically:

1. Build Docker image  
2. Tag image with commit SHA  
3. Push to ECR  
4. Create new task definition revision  
5. Update ECS service  
6. Trigger new deployment  
7. Wait until deployment stabilizes  

---

# Step 7: Validate Deployment

### Check ECS service:

```
ECS → Cluster → Services → node-app-service
```

You should see:

```
Running count: 1
Desired count: 1
Status: ACTIVE
```

### Check Load Balancer:

```
EC2 → Load Balancers → Select ALB
```

Copy the DNS name:

```
http://app-lb-123456.us-east-1.elb.amazonaws.com
```

Open in browser → Your new version should be live.

---

# Troubleshooting

### ❌ *Image not found in ECR*
Check the ECR registry secret:

```
ECR_REGISTRY=123456789012.dkr.ecr.us-east-1.amazonaws.com
```

### ❌ Unauthorized when pushing image
Ensure IAM user has:

```
ecr:BatchCheckLayerAvailability  
ecr:PutImage  
ecr:InitiateLayerUpload  
```

### ❌ ECS service not updating
Ensure correct secret:

```
ECS_SERVICE=your-service-name
```

### ❌ Health check failing
Ensure container exposes the correct port in ECS task:

```
containerPort: 3000
```

---

# Conclusion

With this setup you now have:

✔ Fully automated CI/CD pipeline  
✔ GitHub Actions → ECR → ECS Fargate workflow  
✔ Auto-build + auto-deploy on every push  
✔ Zero-downtime rolling deployments  
✔ Production-ready container delivery  

This workflow is widely used in modern DevOps pipelines and can be extended with:

- Multi-environment pipelines (dev/stage/prod)  
- Canary deployments  
- Terraform IaC  
- Blue/Green deployments with CodeDeploy  

---

**Author:** Your Name  
**Project:** CI/CD Pipeline for ECS Fargate  
