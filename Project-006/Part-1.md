# Deploy React Frontend on AWS Amplify

> Complete step-by-step guide for deploying a React application to AWS Amplify using GitHub as a source repository.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Step 1: Create Ubuntu Instance and Install Dependencies](#step-1-create-ubuntu-instance-and-install-dependencies)
- [Step 2: Create React Application Locally on Ubuntu](#step-2-create-react-application-locally-on-ubuntu)
- [Step 3: Push React App to GitHub Repository](#step-3-push-react-app-to-github-repository)
- [Step 4: Create amplify.yml for AWS Amplify Build Process](#step-4-create-amplifyyml-for-aws-amplify-build-process)
- [Step 5: Deploy Application on AWS Amplify](#step-5-deploy-application-on-aws-amplify)
- [Verification](#verification)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)

---

## Overview

AWS Amplify allows frontend developers to build, deploy, and host React applications easily.  
Once connected to GitHub, Amplify automatically redeploys your app whenever you push new code.

---

## Architecture

```
Local Ubuntu Machine → GitHub Repository → AWS Amplify → Global CDN → Browser
```

---

## Prerequisites

- AWS Account  
- GitHub Account  
- Basic knowledge of React, Git, and Ubuntu  
- Node.js and npm installed  

---

## Step 1: Create Ubuntu Instance and Install Dependencies

### 1.1 Create Ubuntu EC2 Instance
- Allow ports: SSH (22), HTTP (80), HTTPS (443)
- Launch instance and connect to it

### 1.2 Update apt package index

```bash
sudo apt update
```

### 1.3 Install Node.js and npm

```bash
sudo apt install nodejs npm -y
```

### 1.4 Install Git

```bash
sudo apt install git -y
```

---

## Step 2: Create React Application Locally on Ubuntu

### 2.1 Create a new React application

```bash
npx create-react-app my-react-app
cd my-react-app
```

### 2.2 Initialize Git repository

```bash
git init
git add .
git commit -m "initial commit"
```

---

## Step 3: Push React App to GitHub Repository

### 3.1 Create a GitHub repository  
Example:  
```
https://github.com/hanzla6103/react
```

### 3.2 Connect local project with remote GitHub repo

```bash
git remote add origin https://github.com/hanzla6103/react.git
git push -u origin master
```

---

### ⚠️ Note (If You Get Authentication Errors)

If pushing fails, your **GitHub access token may be expired**.

Create a new **Personal Access Token (Classic)** in:

```
GitHub → Settings → Developer Settings → Personal Access Tokens
```

Then update remote origin:

```bash
git remote set-url origin https://hanzla6103:your_access_token@github.com/hanzla6103/react.git
git push -u origin master
```

---

## Step 4: Create amplify.yml for AWS Amplify Build Process

This file tells Amplify how to build your React app.

### 4.1 Create amplify.yml in your GitHub repo root

```yaml
version: 1
frontend:
  phases:
    preBuild:
      commands:
        - yarn install
    build:
      commands:
        - yarn build
  artifacts:
    baseDirectory: build
    files:
      - '**/*'
  cache:
    paths:
      - node_modules/**/*
```

### 4.2 If `amplify.yml` already exists  
Replace its content with the above.

---

## Step 5: Deploy Application on AWS Amplify

### 5.1 Open AWS Amplify Console  
→ **Create New App**  
→ **Host Web App**

### 5.2 Choose GitHub  
- Connect your GitHub account  
- Select your repo (e.g., `react`)  
- Select branch: `master` or `main`  

### 5.3 Amplify detects amplify.yml  
Click **Save & Deploy**

Amplify automatically:

✔ Installs dependencies  
✔ Builds the React app  
✔ Deploys the build folder  
✔ Creates a global CDN endpoint  
✔ Provides a public URL  

### 5.4 Access your App

Click the URL shown in Amplify, for example:

```
https://main.d3asd82kjamplifyapp.com/
```

---

# Verification

### 1. Test the App
Open the Amplify-hosted URL in your browser.

### 2. Make Code Changes

Edit `src/App.js`:

```javascript
<h1>Hello from Amplify Deployment!</h1>
```

Commit the changes:

```bash
git add .
git commit -m "updated UI"
git push
```

Amplify automatically rebuilds and redeploys your site.

### 3. Confirm Deployment
Refresh browser → updated changes should appear.

---

## Troubleshooting

### 🚫 Issue: Build Failed  
Check:

- Missing amplify.yml  
- Yarn not installed (Amplify installs it automatically)  
- Incorrect build path (must be `build`)  

### 🚫 Issue: Git Push Error  
Solution:

```bash
git remote set-url origin https://username:token@github.com/username/repo.git
```

### 🚫 Issue: Amplify Not Detecting React  
Check if you have:

- package.json  
- src/ folder  
- public/ folder  

### 🚫 Issue: App Not Loading  
Verify:

- Build folder exists  
- amplify.yml paths are correct  

---

## Best Practices

### 🔐 Security  
- Never store your GitHub token in code  
- Use `.gitignore` to hide node_modules  

### ⚡ Performance  
- Enable Amplify caching via amplify.yml  
- Minify JS with React build script  

### 🔄 CI/CD  
- Use feature branches  
- Enable pull-request previews  

---

# 🎉 Deployment Completed!

Your React app is now live on AWS Amplify with automatic deployments from GitHub.

---

**Author:** Your Name  
**Repository:** Your GitHub Repo Link  

