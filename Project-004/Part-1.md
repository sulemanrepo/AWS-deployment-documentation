# Deploy Node.js Application on AWS EC2

> Complete guide to deploying a simple Node.js application on an Ubuntu EC2 instance and keeping it running in the background using PM2.

---

## 📋 Table of Contents

- [Prerequisites](#prerequisites)
- [Architecture Overview](#architecture-overview)
- [Step 1: Launch EC2 Instance & Install Node.js](#step-1-launch-ec2-instance--install-nodejs)
- [Step 2: Create Node.js Application](#step-2-create-nodejs-application)
- [Step 3: Run Node.js Application](#step-3-run-nodejs-application)
- [Step 4: Keep Application Running Using PM2](#step-4-keep-application-running-using-pm2)
- [Verification](#verification)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)

---

## Prerequisites

Before beginning, ensure you have:

- ✅ AWS account with EC2 permissions  
- ✅ Basic knowledge of Linux commands  
- ✅ SSH client installed  
- ✅ EC2 key pair (.pem file)

---

## Architecture Overview

```
Internet → Security Group (Port 3000) → EC2 Ubuntu Instance → Node.js Application
```

**Components:**

- **EC2 Instance:** Hosts the Node.js application  
- **Node.js:** JavaScript runtime  
- **PM2:** Process manager to keep the app alive  
- **Port 3000:** Default app port  

---

## Step 1: Launch EC2 Instance & Install Node.js

### 1.1 Create Ubuntu EC2 Instance

1. Open **AWS Console** → **EC2** → **Launch Instance**
2. **AMI:** Ubuntu Server 22.04 LTS  
3. **Instance Type:** t2.micro  
4. **Security Group Rules:**  
   | Type | Protocol | Port | Source | Description |
   |------|----------|------|--------|-------------|
   | SSH | TCP | 22 | Your IP | For SSH access |
   | Custom TCP | TCP | 3000 | 0.0.0.0/0 | Node.js app |

### 1.2 Connect to Your EC2 Instance

```bash
ssh -i "your-key.pem" ubuntu@<EC2-PUBLIC-IP>
```

### 1.3 Update Packages

```bash
sudo apt update
```

### 1.4 Install Node.js & npm

```bash
sudo apt install nodejs npm -y
```

---

## Step 2: Create Node.js Application

### 2.1 Create Application Directory

```bash
mkdir my-node-app
cd my-node-app
```

### 2.2 Create Application File

```bash
touch app.js
nano app.js
```

### 2.3 Add This Sample Code

```javascript
// app.js
const http = require('http');

const hostname = '0.0.0.0';
const port = 3000;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello, World!\n');
});

server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```

**Save file:**  
`CTRL + X` → `Y` → `Enter`

---

## Step 3: Run Node.js Application

### 3.1 Start the App

```bash
node app.js
```

### 3.2 Test From Browser

Open:

```
http://<EC2-PUBLIC-IP>:3000
```

You should see:

```
Hello, World!
```

---

## Step 4: Keep Application Running Using PM2

### 4.1 Install PM2 Globally

```bash
sudo npm install -g pm2
```

### 4.2 Start Application With PM2

```bash
pm2 start app.js
```

### 4.3 Verify That App Keeps Running After Closing SSH

- Close your EC2 terminal  
- Visit:

```
http://<EC2-PUBLIC-IP>:3000
```

If the page loads → **Your app is successfully running in the background!**

---

## Verification

### Checklist

- App accessible on port 3000  
- PM2 process running:

```bash
pm2 list
```

- Node.js file exists:

```bash
cat ~/my-node-app/app.js
```

---

## Troubleshooting

### Issue: App Not Loading in Browser  
✔ Check EC2 security group allows **port 3000**

### Issue: PM2 App Not Running  
```bash
pm2 status
pm2 logs
```

### Issue: Node.js not installed  
```bash
node -v
npm -v
```

---

## Best Practices

### 🔒 Security  
- Do **not** expose port 3000 publicly in production  
- Use **Nginx reverse proxy**  
- Add **SSL** with Let's Encrypt  

### ⚡ Performance  
- Use `pm2 startup` to auto-start on reboot  
- Enable PM2 log rotation:

```bash
pm2 install pm2-logrotate
```

### 📦 Deployment  
- Store dependencies in **package.json**  
- Use CI/CD pipeline for updates  

---

## 🎯 Next Steps

- Integrate Nginx reverse proxy  
- Add PM2 startup script  
  ```bash
  pm2 startup
  pm2 save
  ```
- Deploy a real Node.js app (Express.js)  
- Add domain + HTTPS  

---

## 📝 License

This project is open source under the **MIT License**.

---

**Author:** Your Name  
**Repository:** Your GitHub Repo Link  

