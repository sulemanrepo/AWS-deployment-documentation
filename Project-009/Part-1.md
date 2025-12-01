![Screenshot 1](./images/Architecture.png) 

# Deploy Serverless Image Conversion System Using AWS Lambda & S3

> A complete guide to building an automated image conversion workflow using AWS Lambda and S3.  
> The system converts uploaded images (jpg, jpeg, png, etc.) into PNG format and stores them in a destination bucket.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Step 1: Create S3 Buckets](#step-1-create-s3-buckets)
- [Step 2: Prepare Python Lambda Code](#step-2-prepare-python-lambda-code)
- [Step 3: Create Lambda Function](#step-3-create-lambda-function)
- [Step 4: Add S3 Trigger](#step-4-add-s3-trigger)
- [Step 5: Configure Memory & Timeout](#step-5-configure-memory--timeout)
- [Step 6: Upload Deployment Package](#step-6-upload-deployment-package)
- [Step 7: Test the Workflow](#step-7-test-the-workflow)
- [Monitoring](#monitoring)
- [Conclusion](#conclusion)

---

## Overview

This project demonstrates how to automatically convert uploaded images into another format using AWS services.

The process:

1. User uploads an image to an **input S3 bucket**
2. S3 event triggers an **AWS Lambda function**
3. Lambda converts image into **PNG format**
4. Converted image is saved into an **output S3 bucket**

This solution requires **no servers**, scales automatically, and is ideal for automation workflows.

---

## Architecture

```
User Upload → S3 Input Bucket → S3 Event Trigger → Lambda Function → PNG Output → S3 Output Bucket
```

**Services Used:**
- Amazon S3  
- AWS Lambda  
- IAM  
- CloudWatch  

---

## Step 1: Create S3 Buckets

You will create two buckets.

### 🔹 Input Bucket (original images)
Example:
```
inputimagesforconversion
```

### 🔹 Output Bucket (converted PNG images)
Example:
```
convertedimage
```

### Steps:

1. Open **AWS Console → S3**
2. Click **Create bucket**
3. Name your input bucket → create  
4. Repeat the process for the output bucket  

---

## Step 2: Prepare Python Lambda Code

### 2.1 Prepare environment (optional EC2)

```bash
sudo apt update
sudo apt install python python3-pip -y
```

### 2.2 Create project folder

```bash
mkdir my-lambda-function
cd my-lambda-function
```

### 2.3 Create your Lambda file

```bash
nano lambda_function.py
```

Paste:

```python
import boto3
from PIL import Image
import os

s3 = boto3.client('s3')

def lambda_handler(event, context):
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']

    download_path = f'/tmp/{key}'
    upload_path = f'/tmp/converted.png'

    s3.download_file(bucket, key, download_path)

    img = Image.open(download_path)
    img.save(upload_path, 'PNG')

    s3.upload_file(upload_path, 'convertedimage', f'converted/{key}.png')

    return {
        'statusCode': 200,
        'body': f'{key} converted to PNG successfully'
    }
```

### 2.4 Install Pillow (image processing)

```bash
pip install pillow -t .
```

### 2.5 Zip the deployment package

```bash
zip -r my-lambda-function.zip .
```

Download to local:

```bash
scp -i Downloads/lambda.pem ubuntu@<EC2-IP>:/home/ubuntu/my-lambda-function/my-lambda-function.zip .
```

---

## Step 3: Create Lambda Function

1. Go to **AWS Lambda Console**
2. Click **Create Function**
3. Choose:
   - **Author from scratch**
   - Runtime: **Python 3.x**
   - Function name: `image-converter`
4. Create function

### Attach Execution Role

Create IAM role:

- **AWSLambdaBasicExecutionRole**

Attach it in:

```
Lambda → Configuration → Permissions → Edit Execution Role
```

---

## Step 4: Add S3 Trigger

1. Go to **Lambda Function → Triggers**
2. Click **Add Trigger**
3. Select **S3**
4. Choose your **input bucket**
5. Event type:
   ```
   PUT (Object Created)
   ```
6. Enable → Save

Every time an image is uploaded, Lambda will trigger automatically.

---

## Step 5: Configure Memory & Timeout

Go to:

```
Configuration → General Configuration → Edit
```

Set:

```
Memory: 1024 MB
Timeout: 3 minutes
```

Larger memory improves Pillow processing performance.

---

## Step 6: Upload Deployment Package

1. Go to **Code** tab in Lambda
2. Click **Upload from → .zip**
3. Upload:
```
my-lambda-function.zip
```
4. Save

---

## Step 7: Test the Workflow

### 7.1 Upload any image into input bucket

Drag & drop files into:

```
inputimagesforconversion
```

### 7.2 Check output bucket

Converted PNG files will appear at:

```
convertedimage / converted / <your-file>.png
```

### 7.3 Test with multiple images
Upload 5–20 images to confirm Lambda scales automatically.

---

## Monitoring

Use CloudWatch for logs and debugging:

```
AWS Console → CloudWatch → Logs → image-converter log group
```

You can view:

- Lambda execution logs  
- Errors  
- Processing time  
- Invocation count  

---

## Conclusion

You successfully implemented a **serverless, automated image conversion workflow** using:

- Amazon S3  
- AWS Lambda  
- IAM  
- CloudWatch  

The system converts images into PNG format without managing servers, supports unlimited scaling, and provides fast execution at low cost.

---

**Author:** Your Name  
**Project:** Serverless Image Converter  
