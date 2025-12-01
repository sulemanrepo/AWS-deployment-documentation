# CI/CD Pipeline for Python Application using GitHub → CodePipeline → CodeDeploy → EC2

> A complete, end-to-end guide to deploying a Python application from GitHub to an EC2 instance using CodePipeline, CodeDeploy, and a systemd service — including full workflow scripts and build steps.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Step 1: Create IAM Roles](#step-1-create-iam-roles)
- [Step 2: Launch EC2 Instance](#step-2-launch-ec2-instance)
- [Step 3: Install Python and Dependencies](#step-3-install-python-and-dependencies)
- [Step 4: Clone Application and Setup Virtual Environment](#step-4-clone-application-and-setup-virtual-environment)
- [Step 5: Configure CodeDeploy Application, Deployment Group & CodePipeline](#step-5-configure-codedeploy-application-deployment-group--codepipeline)
- [Step 6: Create Systemd Service](#step-6-create-systemd-service)
- [Step 7: Add appspec.yml and Script Files](#step-7-add-appspecyml-and-script-files)
- [Step 8: Install CodeDeploy Agent](#step-8-install-codedeploy-agent)
- [Folder Structure](#folder-structure)
- [Complete Pipeline Scripts](#complete-pipeline-scripts)
- [Verification](#verification)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)

---

## Overview

This guide will help you create a fully automated CI/CD pipeline that:

✔ Pulls code from GitHub  
✔ Builds using CodePipeline  
✔ Deploys using CodeDeploy  
✔ Automatically restarts your Python app on EC2  

---

## Architecture

```
GitHub → CodePipeline → CodeDeploy → EC2 Instance → systemd service → Python App
```

---

## Prerequisites

- AWS Account  
- GitHub repository  
- EC2 Ubuntu instance  
- Basic Linux & Git knowledge  
- Port **5000** open  

---

## Step 1: Create IAM Roles

### Role 1 → CodeDeploy Service Role  
**Name:** `AWS_CODE_DEPLOY_ROLE`  
**Permissions:**  
- AWSCodeDeployRole  

---

### Role 2 → EC2 IAM Role  
**Name:** `AMAZON_EC2_ROLE_FOR_CODE_DEPLOY`  
**Permissions:**  
- AmazonS3ReadOnlyAccess  
- AWSCodeDeployFullAccess  

Attach this role to EC2.

---

## Step 2: Launch EC2 Instance

1. Create **Ubuntu 22.04** instance  
2. Attach IAM role:  
   → `AMAZON_EC2_ROLE_FOR_CODE_DEPLOY`
3. Add **Security Group Rule**:

| Type | Port | Source | Purpose |
|------|------|--------|----------|
| SSH | 22 | Your IP | Connect |
| Custom TCP | 5000 | 0.0.0.0/0 | Python App Access |

---

## Step 3: Install Python and Dependencies

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y python3 python3-pip python3-venv git
```

---

## Step 4: Clone Application and Setup Virtual Environment

```bash
cd /home/ubuntu
git clone https://github.com/hanzla6103/python-test.git
cd python-test

python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

---

## Step 5: Configure CodeDeploy Application, Deployment Group & CodePipeline

### 5.1 Create CodeDeploy Application
- Compute platform: **EC2**

### 5.2 Create Deployment Group
- Service role: **AWS_CODE_DEPLOY_ROLE**
- Environment: EC2 Instances

### 5.3 Create CodePipeline
- Source: GitHub (v2)
- Build: **Use buildspec.yml**
- Deploy: CodeDeploy

---

## Step 6: Create Systemd Service

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

Enable service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable myapp.service
sudo systemctl start myapp.service
```

---

## Step 7: Add appspec.yml and Script Files

These files must exist **inside your GitHub repository**, not only on server.

Place them in:

```
/python-test/
```

---

## Step 8: Install CodeDeploy Agent

```bash
sudo apt update
sudo apt install ruby wget -y
cd /home/ubuntu

wget https://aws-codedeploy-eu-west-3.s3.eu-west-3.amazonaws.com/latest/install
chmod +x ./install
sudo ./install auto

sudo service codedeploy-agent start
sudo service codedeploy-agent status
```

---

# Folder Structure

```
python-test/
│
├── app.py
├── requirements.txt
├── appspec.yml
│
├── scripts/
│   ├── BeforeInstall.sh
│   ├── AfterInstall.sh
│   ├── ApplicationStart.sh
│   └── validate-service.sh
│
└── buildspec.yml
```

---

# Complete Pipeline Scripts

Below are **all required files** for CodePipeline + CodeDeploy.

---

## 1. `buildspec.yml`

```yaml
version: 0.2

phases:
  install:
    commands:
      - echo "Installing Python dependencies"
      - python3 -m venv venv
      - source venv/bin/activate
      - pip install -r requirements.txt

artifacts:
  files:
    - '**/*'
  name: python-test-artifact
```

---

## 2. `appspec.yml`

```yaml
version: 0.0
os: linux

files:
  - source: /
    destination: /home/ubuntu/python-test
file_exists_behavior: OVERWRITE

hooks:
  BeforeInstall:
    - location: scripts/BeforeInstall.sh
      runas: root
  AfterInstall:
    - location: scripts/AfterInstall.sh
      runas: root
  ApplicationStart:
    - location: scripts/ApplicationStart.sh
      runas: root
  ValidateService:
    - location: scripts/validate-service.sh
      runas: root
```

---

## 3. `scripts/BeforeInstall.sh`

```bash
#!/bin/bash
echo "Stopping existing service..."
sudo systemctl stop myapp.service || true
```

---

## 4. `scripts/AfterInstall.sh`

```bash
#!/bin/bash
echo "Installing dependencies..."
cd /home/ubuntu/python-test
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

---

## 5. `scripts/ApplicationStart.sh`

```bash
#!/bin/bash
echo "Starting application service..."
sudo systemctl start myapp.service
```

---

## 6. `scripts/validate-service.sh`

```bash
#!/bin/bash
echo "Validating service..."
sudo systemctl status myapp.service || exit 1
```

---

# Make Scripts Executable

```bash
chmod +x scripts/*.sh
```

---

# Verification

### Trigger Deployment

Go to **CodePipeline → Release Change**

### Visit App

```
http://<EC2-PUBLIC-IP>:5000
```

### Commit new code  
→ Pipeline should auto-deploy.

---

# Troubleshooting

### View CodeDeploy logs

```bash
sudo tail -f /var/log/aws/codedeploy-agent/codedeploy-agent.log
```

### View lifecycle event logs

```bash
sudo tail -f /opt/codedeploy-agent/deployment-root/deployment-logs/codedeploy-agent-deployments.log
```

### View systemd logs

```bash
sudo journalctl -u myapp.service -f
```

---

# Best Practices

✔ Use `scripts/` folder for hooks  
✔ Use `Nginx` reverse proxy for production  
✔ Use `HTTPS`  
✔ Add CloudWatch alarms  
✔ Use separate dev/staging/prod pipelines  

---

# 🎉 Deployment Completed Successfully

Your pipeline is now fully automated from GitHub → EC2 using CodePipeline + CodeDeploy.  
Every GitHub commit will trigger an automatic deployment.

---

**Author:** Your Name  
**Repository:** Your GitHub Repo Link  

