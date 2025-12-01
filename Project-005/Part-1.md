# CI/CD Pipeline for Python Application on AWS EC2 using CodePipeline & CodeDeploy

> Complete step-by-step guide to deploying a Python application from GitHub to an EC2 instance using AWS CodePipeline, CodeDeploy, virtual environments, and a systemd service.

---

## 📋 Table of Contents

- [Prerequisites](#prerequisites)
- [Architecture Overview](#architecture-overview)
- [Step 1: Create IAM Roles](#step-1-create-iam-roles)
- [Step 2: Launch EC2 Instance](#step-2-launch-ec2-instance)
- [Step 3: Install Python & Dependencies](#step-3-install-python--dependencies)
- [Step 4: Clone Application from GitHub](#step-4-clone-application-from-github)
- [Step 5: Create CodeDeploy Application, Deployment Group & Pipeline](#step-5-create-codedeploy-application-deployment-group--pipeline)
- [Step 6: Create Systemd Service](#step-6-create-systemd-service)
- [Step 7: Create appspec.yml & Hook Scripts](#step-7-create-appspecyml--hook-scripts)
- [Step 8: Install CodeDeploy Agent](#step-8-install-codedeploy-agent)
- [Verification](#verification)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)

---

## Prerequisites

You must have the following:

- ✅ AWS account with IAM, EC2, CodeDeploy, and CodePipeline permissions  
- ✅ GitHub repository with your Python application  
- ✅ Ubuntu EC2 instance  
- ✅ Basic Linux and Git knowledge  

---

## Architecture Overview

```
GitHub → CodePipeline → CodeDeploy → EC2 Instance → Python App Running on Port 5000
```

**Components Used:**

- **GitHub** – stores your application code  
- **AWS CodePipeline** – automates the deployment process  
- **AWS CodeDeploy** – deploys files to EC2 instance  
- **EC2 Instance** – runs the Python application  
- **Systemd Service** – keeps app alive  

---

## Step 1: Create IAM Roles

You need **two IAM roles**:

### 1. AWS_CODE_DEPLOY_ROLE (Service Role)

**Used by:** CodeDeploy  
**Permissions:**  
- AWSCodeDeployRole  
- AmazonEC2FullAccess  
- AmazonS3FullAccess  

### 2. AMAZON_EC2_ROLE_FOR_CODE_DEPLOY (EC2 Instance Role)

**Used by:** EC2 instance  
**Permissions:**
- AmazonS3ReadOnlyAccess  
- AWSCodeDeployFullAccess  

Attach the EC2 role later when launching the EC2 instance.

---

## Step 2: Launch EC2 Instance

### 2.1 Create Ubuntu EC2 Instance

1. Select **Ubuntu Server 22.04 LTS**
2. Choose **t2.micro**
3. Attach **IAM Role:** `AMAZON_EC2_ROLE_FOR_CODE_DEPLOY`
4. Add Security Group rules:

| Type | Protocol | Port | Source | Description |
|------|----------|------|--------|-------------|
| SSH | TCP | 22 | Your IP | Connect via SSH |
| Custom TCP | TCP | 5000 | 0.0.0.0/0 | Application access |

### 2.2 Connect to Instance

```bash
ssh -i "your-key.pem" ubuntu@<EC2-PUBLIC-IP>
```

---

## Step 3: Install Python & Dependencies

Update and install required packages:

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y python3 python3-pip python3-venv git
```

---

## Step 4: Clone Application from GitHub

Navigate to home directory:

```bash
cd /home/ubuntu
```

Clone repository:

```bash
git clone https://github.com/hanzla6103/python-test.git
cd python-test
```

---

### ⚠️ Important Note  
A **virtual environment** is required because Ubuntu uses `apt` for Python packages and not `pip`.

---

### Create Virtual Environment

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

---

## Step 5: Create CodeDeploy Application, Deployment Group & Pipeline

### 5.1 Create CodeDeploy Application

- **Name:** your choice  
- **Compute Platform:** EC2 / On-premises  

### 5.2 Create Deployment Group

- **Name:** your choice  
- **Service Role:** `AWS_CODE_DEPLOY_ROLE`  
- **Environment Configuration:** EC2 Instances  
- **Load Balancer:** Disabled  

### 5.3 Create CodePipeline

- **Name:** your choice  
- **Source Provider:** GitHub (Version 2)  
- **Connect Repository:** python-test  
- **Branch:** main  
- **Build:** **Skip**  
- **Deploy:** CodeDeploy  
- **App Name:** (previous step)  
- **Deployment Group:** (previous step)  

---

## Step 6: Create Systemd Service

Create a systemd file to keep the Python app alive:

```bash
sudo nano /etc/systemd/system/myapp.service
```

Paste:

```ini
[Unit]
Description=Python Test Application
After=network.target

[Service]
User=ubuntu
Group=ubuntu
WorkingDirectory=/home/ubuntu/python-test
ExecStart=/home/ubuntu/python-test/venv/bin/python3 /home/ubuntu/python-test/app.py
Restart=always

[Install]
WantedBy=multi-user.target
```

Reload & enable:

```bash
sudo systemctl daemon-reload
sudo systemctl enable myapp.service
sudo systemctl start myapp.service
```

---

## Step 7: Create appspec.yml & Hook Scripts

### 7.1 Create appspec.yml

```bash
sudo nano appspec.yml
```

Paste:

```yaml
version: 0.0
os: linux
files:
  - source: /
    destination: /home/ubuntu/python-test
file_exists_behavior: OVERWRITE

hooks:
  BeforeInstall:
    - location: /install-dependencies
      runas: root

  AfterInstall:
    - location: /after-install
      runas: root
```

---

### 7.2 Create install-dependencies Script

```bash
sudo nano install-dependencies
```

Paste:

```bash
#!/bin/bash
sudo systemctl stop myapp.service
```

---

### 7.3 Create after-install Script

```bash
sudo nano after-install
```

Paste:

```bash
#!/bin/bash
sudo systemctl restart myapp.service
```

---

### 7.4 Make Scripts Executable

```bash
chmod +x install-dependencies
chmod +x after-install
```

---

## Step 8: Install CodeDeploy Agent

Check if agent exists:

```bash
sudo service codedeploy-agent status
```

Install:

```bash
sudo apt update
sudo apt install ruby wget -y
cd /home/ubuntu

wget https://aws-codedeploy-eu-west-3.s3.eu-west-3.amazonaws.com/latest/install
chmod +x ./install
sudo ./install auto
```

Start agent:

```bash
sudo service codedeploy-agent start
```

---

## Verification

### 1. Trigger pipeline

Go to **AWS CodePipeline → Release Change**

### 2. If successful:

Visit:

```
http://<EC2-PUBLIC-IP>:5000
```

### 3. Commit new code to GitHub

Pipeline should automatically deploy.

---

## Troubleshooting

### ❌ App not updating  
✔ Ensure your GitHub repo contains updated `appspec.yml`, `install-dependencies`, `after-install`.

### ❌ CodeDeploy errors  
Use:

```bash
sudo tail -f /var/log/aws/codedeploy-agent/codedeploy-agent.log
```

### ❌ App not running  
Check service:

```bash
sudo systemctl status myapp.service
```

### ❌ Scripts not running  
Ensure execute permissions:

```bash
chmod +x install-dependencies after-install
```

---

## Best Practices

### 🔒 Security  
- Restrict port 5000 access in production  
- Use Nginx reverse proxy  
- Enable HTTPS with Let's Encrypt  

### ⚙️ DevOps  
- Add **CodeBuild** for automated testing  
- Separate dev/staging/prod pipelines  
- Use S3 artifact storage  

### 📦 Application  
- Keep dependencies minimal  
- Use `requirements.txt` properly  
- Use virtual environment always  

---

## 🎯 Final Notes

Your CI/CD pipeline is now fully automated.  
Whenever you push code to **main**, CodePipeline → CodeDeploy will update the EC2 instance smoothly.

---

**Author:** Your Name  
**Repository:** Your GitHub Repo Link  

