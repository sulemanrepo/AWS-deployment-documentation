# Deploy PHP and Node.js Applications Using Docker on a Single EC2 Instance

> A complete guide to creating Dockerfiles for PHP and Node.js applications, building Docker images, and running both containers simultaneously on a single Ubuntu EC2 instance.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Step 1: Launch EC2 Instance](#step-1-launch-ec2-instance)
- [Step 2: Install Docker](#step-2-install-docker)
- [Step 3: Create PHP Application & Dockerfile](#step-3-create-php-application--dockerfile)
- [Step 4: Build & Run PHP Docker Container](#step-4-build--run-php-docker-container)
- [Step 5: Create Node.js Application & Dockerfile](#step-5-create-nodejs-application--dockerfile)
- [Step 6: Build & Run Node.js Docker Container](#step-6-build--run-nodejs-docker-container)
- [Verification](#verification)
- [Conclusion](#conclusion)

---

## Overview

This guide shows how to:

✔ Create Dockerfiles for **both PHP and Node.js apps**  
✔ Build Docker images  
✔ Run both Docker containers on a **single EC2 instance**  
✔ Expose PHP on port **80** and Node.js on port **3000**

This setup is ideal for testing multi-container workloads on a single server.

---

## Architecture

```
User → EC2 Instance → Docker Engine
      ├── PHP Container   (Port 80)
      └── Node.js Container (Port 3000)
```

---

# Step 1: Launch EC2 Instance

1. Open **AWS Console → EC2 → Launch Instance**
2. Choose:
   - AMI: **Ubuntu Server 22.04**
   - Instance type: t2.micro
3. **Security Group rules:**

| Type        | Port | Source    | Description         |
|-------------|------|-----------|---------------------|
| SSH         | 22   | Your IP   | Instance access     |
| HTTP        | 80   | Anywhere  | PHP container access |
| Custom TCP  | 3000 | Anywhere  | Node.js container access |

4. Launch instance and connect:

```bash
ssh -i key.pem ubuntu@<EC2-PUBLIC-IP>
```

---

# Step 2: Install Docker

### Update packages

```bash
sudo apt update
```

### Install Docker Engine

```bash
sudo apt-get install docker.io -y
```

### Start, enable & verify Docker

```bash
sudo systemctl start docker
sudo systemctl enable docker
sudo systemctl status docker
```

---

# Step 3: Create PHP Application & Dockerfile

Navigate to project directory:

```bash
mkdir my-project
cd my-project
```

Create PHP folder:

```bash
mkdir php-app
cd php-app
```

### Create Dockerfile

```bash
sudo nano dockerfile
```

Paste:

```dockerfile
FROM php:7.4-apache
COPY . /var/www/html/
EXPOSE 80
CMD ["apache2-foreground"]
```

### Create PHP file

```bash
echo "<?php phpinfo(); ?>" > index.php
```

---

# Step 4: Build & Run PHP Docker Container

### Add user to Docker group (important)

```bash
sudo usermod -aG docker $USER
```

Log out & log back in, then verify:

```bash
groups
```

### Build PHP image

```bash
docker build -t php-app .
```

### Run PHP container

```bash
docker run -d -p 80:80 --name php-container php-app
```

---

# Step 5: Create Node.js Application & Dockerfile

Go back to project folder:

```bash
cd ..
mkdir node-app
cd node-app
```

### Create Dockerfile

```bash
sudo nano dockerfile
```

Paste:

```dockerfile
FROM node:14
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "app.js"]
```

### Create `package.json`

```bash
sudo nano package.json
```

Paste:

```json
{
  "name": "node-app",
  "version": "1.0.0",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "express": "^4.17.1"
  }
}
```

### Create `app.js`

```bash
sudo nano app.js
```

Paste:

```javascript
const express = require('express');
const app = express();
const port = 3000;

app.get('/', (req, res) => {
  res.send('Hello World!');
});

app.listen(port, () => {
  console.log(`App listening at http://localhost:${port}`);
});
```

---

# Step 6: Build & Run Node.js Docker Container

### Build the Node.js image

```bash
docker build -t node-app .
```

### Run the Node.js container

```bash
docker run -d -p 3000:3000 --name node-container node-app
```

---

# Verification

### ✔ PHP App
Open in browser:

```
http://<EC2-PUBLIC-IP>
```

You should see the PHP info page.

---

### ✔ Node.js App
Open:

```
http://<EC2-PUBLIC-IP>:3000
```

You should see:

```
Hello World!
```

---

# Conclusion

You have successfully:

- Installed Docker on EC2  
- Created Dockerfiles for PHP and Node.js  
- Built two Docker images  
- Ran both applications on a single EC2 instance  
- Exposed services on ports 80 and 3000  

This setup is perfect for lightweight multi-service deployments or testing.

---

**Author:** Your Name  
**Project:** PHP + Node.js Docker Deployment  
