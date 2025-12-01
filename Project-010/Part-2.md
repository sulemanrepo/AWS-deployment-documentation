# Docker Load Balancing for PHP & Node.js Apps Using Nginx and Docker Compose

> This section continues from the previous project and implements **Nginx load balancing** across both Docker containers (PHP & Node.js) running on a single EC2 instance.  
> We will configure Nginx as a reverse proxy load balancer and use Docker Compose to run all services together.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Step 1: Create Nginx Configuration](#step-1-create-nginx-configuration)
- [Step 2: Create Docker Compose File](#step-2-create-docker-compose-file)
- [Step 3: Install Docker Compose & Start Services](#step-3-install-docker-compose--start-services)
- [Step 4: Verify Load Balancer](#step-4-verify-load-balancer)
- [Conclusion](#conclusion)

---

## Overview

In this part, you will:

✔ Create an Nginx configuration to load balance between PHP and Node.js containers  
✔ Build an Nginx container  
✔ Link all three containers (PHP, Node.js, Nginx) using Docker Compose  
✔ Access both applications through a single EC2 public IP  

---

## Architecture

```
                      ┌──────────────────────┐
                      │      Nginx LB         │
                      │   (Port 80 → LB)      │
                      └──────────┬────────────┘
                                 │
         ┌───────────────────────┼────────────────────────┐
         │                                               │
 ┌───────────────┐                                ┌───────────────┐
 │ PHP Container  │                                │ NodeJS Container │
 │ (Port 80)      │                                │ (Port 3000)       │
 └───────────────┘                                └───────────────┘
```

Nginx will forward traffic alternately to the PHP app and Nodejs app.

---

# Step 1: Create Nginx Configuration

Navigate to your **my-project** directory:

```bash
cd my-project
mkdir nginx
cd nginx
```

### Create the Nginx config file:

```bash
sudo nano nginx.conf
```

Paste:

```nginx
events {
    worker_connections 1024;
}

http {
    upstream backend {
        server php-container:80 weight=1;
        server node-container:3000 weight=1;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

---

### Create a Dockerfile for Nginx:

```bash
sudo nano dockerfile
```

Paste:

```dockerfile
FROM nginx:latest
COPY nginx.conf /etc/nginx/nginx.conf
```

### Build the Nginx image:

```bash
docker build -t nginx-loadbalancer .
```

---

# Step 2: Create Docker Compose File

Navigate back to project root:

```bash
cd ..
```

Create the Docker Compose file:

```bash
sudo nano docker-compose.yml
```

Paste:

```yaml
version: '3'

services:
  php-app:
    build: ./php-app
    container_name: php-container
    ports:
      - "8080:80"

  node-app:
    build: ./node-app
    container_name: node-container
    ports:
      - "3000:3000"

  nginx:
    build: ./nginx
    container_name: nginx-container
    ports:
      - "80:80"
    depends_on:
      - php-app
      - node-app
```

---

# Step 3: Install Docker Compose & Start Services

Before continuing, remove any existing containers manually or using:

```bash
docker rm -f php-container node-container nginx-container
```

### Install Docker Compose:

```bash
sudo apt-get update
sudo systemctl start docker
sudo systemctl enable docker
```

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

```bash
sudo chmod +x /usr/local/bin/docker-compose
```

### Start all services:

```bash
docker-compose up -d
```

This will:

- Build PHP container  
- Build Node.js container  
- Build Nginx container  
- Start and connect all containers  

---

# Step 4: Verify Load Balancer

Open your browser:

```
http://<EC2-PUBLIC-IP>
```

Now refresh the page multiple times.

You should see alternating responses:

✔ PHP app (phpinfo)  
✔ Node.js app (“Hello World!”)  

This confirms:

- Nginx load balancer is working  
- PHP container is running  
- Node.js container is running  
- All containers linked correctly through Docker Compose  

---

# Conclusion

You have successfully implemented:

✔ PHP & Node.js Docker containers  
✔ Nginx reverse proxy load balancer  
✔ Docker Compose orchestration  
✔ Multi-container application running on a single EC2 instance  

Your setup is now production-ready and can be extended with:

- SSL via Nginx  
- Horizontal scaling using ECS or Kubernetes  
- Logging & monitoring  

---

**Author:** Your Name  
**Project:** Docker Multi-Container with Load Balancing  
