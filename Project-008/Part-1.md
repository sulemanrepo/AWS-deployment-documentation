# Deploy Python Application on AWS Lambda with API Gateway Endpoint

> A complete step-by-step guide to creating a Python Lambda function, uploading code, and exposing it publicly through API Gateway.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Step 1: Prepare Ubuntu Instance](#step-1-prepare-ubuntu-instance)
- [Step 2: Create Python Lambda Function Code](#step-2-create-python-lambda-function-code)
- [Step 3: Download Deployment Zip File](#step-3-download-deployment-zip-file)
- [Step 4: Create Lambda Function in AWS](#step-4-create-lambda-function-in-aws)
- [Step 5: Create API Gateway Endpoint](#step-5-create-api-gateway-endpoint)
- [Testing the Endpoint](#testing-the-endpoint)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)

---

## Overview

AWS Lambda allows you to deploy serverless Python applications without managing servers.  
API Gateway provides a public HTTP endpoint to invoke your Lambda function.

This guide walks you through:

✔ Writing Lambda code  
✔ Packaging and uploading a `.zip` file  
✔ Creating a Lambda function  
✔ Creating REST API in API Gateway  
✔ Deploying and testing the endpoint  

---

## Architecture

```
Client → API Gateway → Lambda Function → Response
```

---

## Prerequisites

- AWS Account  
- EC2 Ubuntu instance (optional but used here for packaging)  
- Basic Python knowledge  
- SSH access  

---

# Step 1: Prepare Ubuntu Instance

Connect to your Ubuntu instance and update packages:

```bash
sudo apt update
sudo apt install python python3-pip -y
```

---

# Step 2: Create Python Lambda Function Code

### 2.1 Create project directory

```bash
mkdir my-lambda-function
cd my-lambda-function
```

### 2.2 Create your Lambda function file

```bash
sudo nano lambda-function.py
```

Paste the following code:

```python
def lambda_handler(event, context):
    return {
        'statusCode': 200,
        'body': 'Hello from Lambda!'
    }
```

### 2.3 Install zip utility

```bash
sudo apt install zip -y
```

### 2.4 Zip your Lambda function

```bash
zip -r my-lambda-function.zip .
```

---

# Step 3: Download Deployment Zip File to Local Machine

Run this from your **local computer terminal**:

```bash
scp -i Downloads/lambda.pem ubuntu@<EC2-IP>:/home/ubuntu/my-lambda-function/my-lambda-function.zip .
```

Now your deployment package is on your local machine.

---

# Step 4: Create Lambda Function in AWS

### 4.1 Create Function

1. Go to **AWS Lambda Console**
2. Click **Create Function**
3. Choose:
   - **Author from scratch**
   - Function name: `my-lambda-function`
   - Runtime: **Python 3.x**
4. Click **Create Function**

---

### 4.2 Upload Deployment Package

1. Open your Lambda function  
2. Go to **Code** tab  
3. Click **Upload from → .zip file**  
4. Upload `my-lambda-function.zip`  

---

### 4.3 Create IAM Role for Lambda

Create a role:

- **AWSLambdaBasicExecutionRole**

Attach the role to Lambda:

- Go to **Configuration → Permissions**
- Edit Execution Role → Select IAM role created

---

# Step 5: Create API Gateway Endpoint

This step exposes your Lambda function publicly.

---

## 5.1 Create REST API

1. Open **API Gateway Console**
2. Click **Create API**
3. Choose **REST API**
4. Click **Build**

---

## 5.2 Create Resource

1. Click **Actions → Create Resource**
2. Name: `myresource`
3. Resource Path: `/myresource`

---

## 5.3 Create Method

1. Click on your resource `/myresource`
2. Click **Actions → Create Method**
3. Choose **GET**
4. Integration Type: **Lambda Function**
5. Select your region
6. Enter Lambda function name
7. Save
8. Approve permissions popup

---

## 5.4 Deploy the API

1. Click **Actions → Deploy API**
2. Choose **New Stage**
3. Stage name: `prod`
4. Deploy

After deployment, you will see:

```
Invoke URL: https://<api-id>.execute-api.<region>.amazonaws.com/prod/myresource
```

This is your **public endpoint**.

---

# Testing the Endpoint

Open this in your browser:

```
https://<api-id>.execute-api.<region>.amazonaws.com/prod/myresource
```

Expected output:

```
Hello from Lambda!
```

---

# Troubleshooting

### ❌ Lambda returns "Internal Server Error"
Check CloudWatch logs:

```
Lambda → Monitor → View logs in CloudWatch
```

### ❌ API shows "Missing Authentication Token"
Ensure you are hitting:

```
/prod/myresource
```

### ❌ Changes not applied
Redeploy API:

```
API Gateway → Actions → Deploy API → prod
```

---

# Best Practices

✔ Use environment variables via Lambda console  
✔ Add logging with CloudWatch  
✔ Use Python virtual environments if dependencies are required  
✔ Protect APIs with API Keys or Cognito  
✔ Use Lambda Layers for shared Python packages  

---

# 🎉 Deployment Complete!

You have successfully:

- Created a Python Lambda function  
- Packaged and uploaded code  
- Built a REST API in API Gateway  
- Generated a working public endpoint  

---

**Author:** Your Name  
**Repository:** Your GitHub Repo Link  

