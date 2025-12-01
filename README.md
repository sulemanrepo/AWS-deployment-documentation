# AWS Cloud & DevOps Deployment Projects  
This repository contains a complete collection of hands-on AWS and DevOps projects demonstrating real-world deployment, automation, containerization, configuration management, and CI/CD experience. Every project includes production-grade documentation, commands, and architecture patterns.

---

## 👨‍💻 About Me  
I am a DevOps & Cloud Engineer with **2+ years of hands-on AWS experience**, working with modern cloud-native environments, CI/CD pipelines, automation tools, and scalable application deployments.

### 🧩 Skills & Technologies  
**Cloud Services:** EC2, Lightsail, ECS Fargate, ECR, Lambda, API Gateway, RDS, S3, Load Balancer, VPC, IAM, CloudWatch, Route53, CloudFront  
**DevOps:** GitHub Actions CI/CD, Ansible, Docker, Docker Compose, container orchestration  
**Programming / Runtime:** Node.js, Python, PHP, Linux shell  
**Web Servers:** Apache, Nginx  
**Security:** SSL (Certbot), IAM Roles, Secrets Manager, Parameter Store  
**Other:** Git, Bash scripting, YAML config, systemd automation  

---

# 📂 Project Summaries

Below are professional 3–4 line descriptions for each project in this repository.

---

## **📁 Project-001 — Deploy WordPress on AWS EC2 (LAMP + Domain + SSL)**
This project covers deploying WordPress on an Ubuntu EC2 instance using the LAMP stack. It includes MySQL configuration, Apache setup, WordPress file deployment, domain mapping through Route53, and enabling HTTPS using Let’s Encrypt (Certbot). Full troubleshooting steps and best practices for database connectivity and SSL renewal are also included.

---

## **📁 Project-002 — WordPress Migration & Backup on AWS Lightsail (Part 1 & Part 2)**  
### **Part 1 — Migrating WordPress to AWS Lightsail using All-in-One WP Migration**  
A complete migration of an existing WordPress site into AWS Lightsail using the All-in-One WP Migration plugin. Covers Lightsail instance provisioning, admin access, plugin installation, upload size limit handling, importing .wpress backups, and post-migration verification with URLs, permalinks, and SSL recommendations.

### **Part 2 — Manual WordPress Backup from AWS Lightsail (Files + Database)**  
This part documents how to manually export the WordPress directory and database from a Lightsail server. It includes compressing site files, generating SQL backups, downloading them via SCP, verifying integrity, and applying best practices for secure backup handling. Troubleshooting steps for MySQL dumps, permissions, and storage issues are provided.

---

## **📁 Project-003 — Deploy Python Flask Application on AWS EC2 (Virtual Env + systemd)**
This project demonstrates deploying a production-ready Python Flask application on an Ubuntu EC2 instance. It covers preparing the server environment, creating a virtual environment, installing dependencies, and running the Flask app on port 5000. A full systemd service is configured to keep the application running in the background with auto-restart and boot persistence. Includes testing, troubleshooting, and best practices for stable Flask deployments on AWS.

---

## **📁 Project-004 — Deploy Node.js Application on AWS EC2 Using PM2**
This project demonstrates deploying a Node.js application on an Ubuntu EC2 instance and running it reliably using PM2 as a process manager. It includes installing Node.js, configuring a simple HTTP server, exposing port 3000, and ensuring the application runs continuously even after the SSH session ends. The documentation also covers verification, troubleshooting, and best practices like reverse proxying with Nginx and enabling PM2 startup on reboot.

---

## **📁 Project-005 — CI/CD Pipeline for Python Application (GitHub → CodePipeline → CodeDeploy → EC2)**
This project delivers a complete end-to-end CI/CD pipeline for deploying a Python application from GitHub to an EC2 instance using CodePipeline and CodeDeploy. It includes automated build steps, lifecycle hook scripts, virtual environment setup, and systemd service integration for zero-downtime deployments. The workflow builds Docker-free artifacts, installs dependencies, updates the EC2 service, and validates deployments with fully automated rollback capability.

---

## **📁 Project-006 — Deploy React Frontend on AWS Amplify (CI/CD with GitHub)**
This project demonstrates the end-to-end deployment of a React application using AWS Amplify with GitHub integration. It covers creating a local React project, pushing it to GitHub, configuring Amplify build settings through `amplify.yml`, and enabling automatic deployments on every code push. The solution delivers a globally distributed, CDN-backed frontend with seamless CI/CD, caching optimizations, and secure GitHub-based version control integration.

---

## **📁 Project-007 — Deploy Node.js & PHP Applications on AWS Elastic Beanstalk**
This project covers the full deployment workflow for running both Node.js and PHP applications on AWS Elastic Beanstalk. It includes building and testing both applications on an Ubuntu instance, creating systemd services, packaging the code into deployment-ready ZIP bundles, and deploying them to separate Elastic Beanstalk environments. The documentation also explains environment setup, troubleshooting, verification steps, and best practices for scalable, production-ready deployments.

---

## **📁 Project-008 — Deploy Python Application on AWS Lambda with API Gateway**
This project demonstrates the complete workflow of deploying a Python-based serverless application on AWS Lambda and exposing it publicly through API Gateway. It includes writing and packaging the Lambda function, creating the deployment ZIP, uploading it to Lambda, configuring IAM permissions, and building a REST API in API Gateway. The documentation also covers endpoint testing, troubleshooting, and best practices for secure and scalable serverless deployments.

---

## **📁 Project-009 — Serverless Image Conversion System Using AWS Lambda & S3**
This project showcases a fully serverless image-processing pipeline using AWS Lambda and Amazon S3. When a user uploads an image to the input bucket, an S3 event triggers a Lambda function that converts the image into PNG format using Pillow and stores the output in a destination S3 bucket. The workflow is scalable, automated, and ideal for lightweight image-processing tasks without managing servers.

---

## **📁 Project-010 — Dockerized PHP & Node.js Deployment with Nginx Load Balancing**

### **Part 1 — Deploy PHP & Node.js Applications on a Single EC2 Instance Using Docker**  
This part covers containerizing both PHP and Node.js applications using Docker, building their respective Dockerfiles, and running both workloads simultaneously on an Ubuntu EC2 instance. The PHP container runs on **port 80**, while the Node.js application is exposed on **port 3000**. It includes Docker installation, image builds, container execution, and verification steps to ensure both applications operate independently on a single server.

### **Part 2 — Implementing Nginx Load Balancer Using Docker Compose**  
This extension introduces **Nginx as a reverse proxy load balancer** to route traffic between PHP and Node.js containers. It includes creating a custom Nginx config, building the Nginx image, and orchestrating all services using **Docker Compose**. The final architecture forwards all traffic through port **80**, alternating responses between both apps, delivering a complete multi-container environment with centralized access and scalable design.

---

## **📁 Project-011 — Deploy Docker Containers on AWS ECS Fargate with ALB**

### **Deploying Node.js & PHP Containers on ECS Fargate with Application Load Balancer**
This project demonstrates how to containerize applications, push images to **Amazon ECR Public**, and deploy them on **AWS ECS Fargate** using task definitions and services. The Node.js app is exposed via an **Application Load Balancer (ALB)**, while the PHP app can be deployed similarly for independent service scaling.

### **Core Workflow Covered**
The documentation walks through creating public ECR repositories, authenticating Docker, building & pushing images, configuring task definitions, provisioning a Fargate cluster, attaching an ALB with IP-target groups, and deploying services. The final setup enables fully managed, serverless, auto-scaled container hosting with clean traffic routing through the ALB.


---

## **📁 Project-012 — GitHub Actions CI/CD Pipeline for AWS ECS Fargate Deployment**

### **End-to-End CI/CD for Containerized Applications on ECS Fargate**
This project delivers a complete, production-ready CI/CD workflow that builds Docker images, pushes them to Amazon ECR, updates ECS task definitions, and triggers zero-downtime deployments on AWS ECS Fargate — all automated through GitHub Actions.

### **Pipeline Workflow Overview**
The documentation covers creation of an ECR repository, IAM user setup for GitHub Actions, secure GitHub repository secrets, ECS cluster preparation, task definition templating, and the full deploy.yml pipeline.  
Every code push to the *main* branch automatically builds the Docker image, tags it with the commit SHA, publishes it to ECR, and rolls out a new revision to ECS Fargate.  
This provides a robust, scalable CI/CD system used in modern DevOps environments for seamless container deployments.


---

## **📁 Project-013 — Ansible Playbook for Remote Package Installation & Updates**

### **Automated Package Management Using Ansible (Master → Node Setup)**
This project documents the full workflow of setting up an Ansible control node (Master) to manage a remote Node server over passwordless SSH.  
It includes installing Ansible, generating SSH keys, configuring inventory, and creating playbooks to install and update packages across remote machines.

### **Included Deliverables**
- Master & Node EC2 instance setup  
- SSH key-based authentication configuration  
- Ansible inventory file creation  
- Playbook for updating package cache, installing software, and performing full upgrades  
- Separate playbook for installing cURL only  
- Verification steps to confirm remote execution works successfully

This provides a foundational automation workflow that can be expanded to manage larger infrastructure efficiently.
