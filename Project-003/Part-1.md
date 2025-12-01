# Deploy Python Flask Application on AWS EC2

> Complete guide to deploying a simple Python Flask web application on an Ubuntu EC2 instance and making it accessible online.

---

## 📋 Table of Contents

- [Prerequisites](#prerequisites)
- [Architecture Overview](#architecture-overview)
- [Step 1: Launch EC2 Instance & Install Dependencies](#step-1-launch-ec2-instance--install-dependencies)
- [Step 2: Create Flask Application](#step-2-create-flask-application)
- [Step 3: Test Flask Application](#step-3-test-flask-application)
- [Step 4: Create Systemd Service](#step-4-create-systemd-service)
- [Step 5: Manage Application Service](#step-5-manage-application-service)
- [Verification](#verification)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)

---

## Prerequisites

Before starting, ensure you have:

- ✅ AWS account with EC2 access
- ✅ Basic understanding of Linux commands
- ✅ SSH client installed (Terminal/PuTTY)
- ✅ EC2 key pair (.pem file)

---

## Architecture Overview

```
Internet → EC2 Security Group (Port 5000) → Ubuntu EC2 Instance → Flask App (Python)
```

**Components:**
- **EC2 Instance**: Ubuntu Server hosting the application
- **Flask**: Python web framework
- **Systemd**: Service manager to keep app running
- **Port 5000**: Default Flask application port

---

## Step 1: Launch EC2 Instance & Install Dependencies

### 1.1 Create Ubuntu EC2 Instance

1. Go to **AWS Console** → **EC2** → **Launch Instance**
2. **Choose AMI**: Ubuntu Server 22.04 LTS
3. **Instance Type**: t2.micro (Free tier eligible)
4. **Configure Security Group**:
   - SSH (Port 22) - Your IP
   - Custom TCP (Port 5000) - 0.0.0.0/0 (Anywhere)
5. **Create/Select Key Pair** and download it
6. **Launch Instance**

### 1.2 Configure Security Group

| Type | Protocol | Port Range | Source | Description |
|------|----------|------------|--------|-------------|
| SSH | TCP | 22 | Your IP | SSH access |
| Custom TCP | TCP | 5000 | 0.0.0.0/0 | Flask application |

> ⚠️ **Important**: Make sure port 5000 is open in your security group, otherwise you won't be able to access your application from the browser.

### 1.3 Connect to EC2 Instance

```bash
ssh -i "path/to/your-key.pem" ubuntu@<EC2-Public-IP>
```

**Example:**

```bash
ssh -i "Downloads/flask-key.pem" ubuntu@54.123.45.67
```

### 1.4 Update System Packages

```bash
sudo apt update
```

### 1.5 Install Python and Pip

```bash
sudo apt install python3 python3-pip -y
```

### 1.6 Install Flask (System-wide - Optional)

```bash
sudo apt install python3-flask -y
```

> 📝 **Note**: We'll also install Flask in a virtual environment later for better dependency management.

---

## Step 2: Create Flask Application

### 2.1 Create Application Directory

```bash
mkdir ~/myapp
cd ~/myapp
```

### 2.2 Set Up Virtual Environment

Create and activate a Python virtual environment:

```bash
python3 -m venv venv
source venv/bin/activate
```

You should see `(venv)` prefix in your terminal:

```
(venv) ubuntu@ip-172-31-xx-xx:~/myapp$
```

### 2.3 Install Flask in Virtual Environment

```bash
pip install Flask
```

### 2.4 Create Python Application File

```bash
touch app.py
nano app.py
```

### 2.5 Add Flask Application Code

Copy and paste the following code:

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello, World!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

**Save and exit:**
- Press `Ctrl + X`
- Press `Y` to confirm
- Press `Enter` to save

### 2.6 Verify File Contents

```bash
cat app.py
```

---

## Step 3: Test Flask Application

### 3.1 Run Flask Application Manually

Make sure you're in the virtual environment and run:

```bash
python app.py
```

**Expected output:**

```
 * Serving Flask app 'app'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:5000
 * Running on http://172.31.xx.xx:5000
Press CTRL+C to quit
```

### 3.2 Test from Browser

Open your browser and navigate to:

```
http://<EC2-Public-IP>:5000
```

**Example:**

```
http://54.123.45.67:5000
```

You should see: **"Hello, World!"**

### 3.3 Stop the Application

Press `Ctrl + C` in the terminal to stop the Flask app.

---

## Step 4: Create Systemd Service

To keep your application running in the background and automatically restart on failure or reboot, we'll create a systemd service.

### 4.1 Create Service File

```bash
sudo nano /etc/systemd/system/myapp.service
```

### 4.2 Add Service Configuration

> ⚠️ **CRITICAL**: Use the **complete absolute paths** for `WorkingDirectory` and `ExecStart`. Relative paths will cause errors!

```ini
[Unit]
Description=My Python Flask Application
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/myapp
ExecStart=/home/ubuntu/myapp/venv/bin/python /home/ubuntu/myapp/app.py
Restart=always

[Install]
WantedBy=multi-user.target
```

**Configuration breakdown:**
- `Description`: Service description
- `After=network.target`: Start after network is available
- `User=ubuntu`: Run as ubuntu user
- `WorkingDirectory`: Full path to app directory
- `ExecStart`: Full path to Python in venv + Full path to app.py
- `Restart=always`: Restart on failure
- `WantedBy=multi-user.target`: Enable on multi-user runlevel

**Save and exit** (`Ctrl + X` → `Y` → `Enter`)

### 4.3 Verify Service File

```bash
sudo cat /etc/systemd/system/myapp.service
```

---

## Step 5: Manage Application Service

### 5.1 Reload Systemd Daemon

After creating the service file, reload systemd to recognize it:

```bash
sudo systemctl daemon-reload
```

### 5.2 Start the Service

```bash
sudo systemctl start myapp.service
```

### 5.3 Check Service Status

```bash
sudo systemctl status myapp.service
```

**Expected output (successful):**

```
● myapp.service - My Python Flask Application
     Loaded: loaded (/etc/systemd/system/myapp.service; disabled; vendor preset: enabled)
     Active: active (running) since Mon 2024-12-01 10:30:15 UTC; 5s ago
   Main PID: 12345 (python)
      Tasks: 1 (limit: 1131)
     Memory: 25.3M
        CPU: 234ms
     CGroup: /system.slice/myapp.service
             └─12345 /home/ubuntu/myapp/venv/bin/python /home/ubuntu/myapp/app.py
```

### 5.4 Enable Service to Start on Boot

```bash
sudo systemctl enable myapp.service
```

### 5.5 Common Service Commands

```bash
# Start service
sudo systemctl start myapp.service

# Stop service
sudo systemctl stop myapp.service

# Restart service
sudo systemctl restart myapp.service

# Check status
sudo systemctl status myapp.service

# View logs
sudo journalctl -u myapp.service -f

# Disable auto-start on boot
sudo systemctl disable myapp.service
```

---

## Verification

### ✅ Checklist

1. **Service is running:**
   ```bash
   sudo systemctl status myapp.service
   ```
   Should show `active (running)`

2. **Application is accessible:**
   - Open browser: `http://<EC2-Public-IP>:5000`
   - Should display: "Hello, World!"

3. **Service auto-starts on reboot:**
   ```bash
   sudo reboot
   # Wait for instance to restart, then check:
   sudo systemctl status myapp.service
   ```

4. **Check application logs:**
   ```bash
   sudo journalctl -u myapp.service -n 50
   ```

---

## Troubleshooting

### Issue: Cannot access application from browser

**Symptoms:**
- Browser shows "Connection refused" or "Unable to connect"

**Solutions:**

1. **Check security group:**
   ```bash
   # Verify port 5000 is open in EC2 Security Group
   # Allow 0.0.0.0/0 on TCP port 5000
   ```

2. **Check if service is running:**
   ```bash
   sudo systemctl status myapp.service
   ```

3. **Check if port is listening:**
   ```bash
   sudo netstat -tulpn | grep 5000
   # or
   sudo ss -tulpn | grep 5000
   ```

### Issue: Service fails to start

**Symptoms:**
- Status shows `failed` or `inactive (dead)`

**Solutions:**

1. **Check service logs:**
   ```bash
   sudo journalctl -u myapp.service -n 50 --no-pager
   ```

2. **Verify paths in service file:**
   ```bash
   # Check if paths exist
   ls -la /home/ubuntu/myapp/venv/bin/python
   ls -la /home/ubuntu/myapp/app.py
   ```

3. **Test running manually:**
   ```bash
   cd ~/myapp
   source venv/bin/activate
   python app.py
   ```

4. **Check for syntax errors:**
   ```bash
   cd ~/myapp
   source venv/bin/activate
   python -m py_compile app.py
   ```

### Issue: Service starts but immediately stops

**Solutions:**

1. **Check for port conflicts:**
   ```bash
   sudo lsof -i :5000
   ```

2. **Verify Flask is installed in venv:**
   ```bash
   /home/ubuntu/myapp/venv/bin/pip list | grep Flask
   ```

3. **Check file permissions:**
   ```bash
   ls -la ~/myapp/
   ```

### Issue: "ModuleNotFoundError: No module named 'flask'"

**Solution:**

Ensure Flask is installed in the virtual environment:

```bash
cd ~/myapp
source venv/bin/activate
pip install Flask
```

### Issue: Permission denied errors

**Solution:**

```bash
# Fix ownership
sudo chown -R ubuntu:ubuntu /home/ubuntu/myapp

# Fix permissions
chmod +x /home/ubuntu/myapp/app.py
```

---

## Best Practices

### 🔒 Security

1. **Restrict SSH access:**
   - Change security group SSH rule from `0.0.0.0/0` to your specific IP

2. **Use environment variables:**
   ```python
   import os
   app.config['SECRET_KEY'] = os.environ.get('SECRET_KEY', 'dev-key')
   ```

3. **Don't run as root:**
   - Service file already uses `User=ubuntu`

4. **Use HTTPS in production:**
   - Set up Nginx as reverse proxy
   - Install SSL certificate (Let's Encrypt)

### ⚡ Performance

1. **Use production WSGI server:**
   ```bash
   pip install gunicorn
   ```
   
   Update service file:
   ```ini
   ExecStart=/home/ubuntu/myapp/venv/bin/gunicorn -w 4 -b 0.0.0.0:5000 app:app
   ```

2. **Use Nginx as reverse proxy:**
   - Better static file serving
   - Load balancing
   - SSL termination

### 🛠️ Development

1. **Use debug mode for development:**
   ```python
   if __name__ == '__main__':
       app.run(host='0.0.0.0', port=5000, debug=True)
   ```

2. **Keep dependencies updated:**
   ```bash
   pip list --outdated
   pip install --upgrade Flask
   ```

3. **Use requirements.txt:**
   ```bash
   pip freeze > requirements.txt
   ```

### 📊 Monitoring

1. **View live logs:**
   ```bash
   sudo journalctl -u myapp.service -f
   ```

2. **Check resource usage:**
   ```bash
   htop
   # or
   top
   ```

3. **Monitor disk space:**
   ```bash
   df -h
   ```

---

## 🎯 Next Steps

After successfully deploying your Flask app:

1. **Add more routes** to your Flask application
2. **Set up Nginx** as reverse proxy
3. **Install SSL certificate** with Let's Encrypt
4. **Add database** (PostgreSQL/MySQL)
5. **Implement logging** for better debugging
6. **Set up CI/CD** pipeline for automated deployments
7. **Configure monitoring** (CloudWatch, Prometheus)

---

## 📚 Additional Resources

- [Flask Documentation](https://flask.palletsprojects.com/)
- [Systemd Service Documentation](https://www.freedesktop.org/software/systemd/man/systemd.service.html)
- [AWS EC2 User Guide](https://docs.aws.amazon.com/ec2/)
- [Python Virtual Environments](https://docs.python.org/3/library/venv.html)

---

## 📝 License

This project is open source and available under the [MIT License](LICENSE).

## 🤝 Contributing

Contributions, issues, and feature requests are welcome!

---

**Author:** [Your Name](https://github.com/sulemanrepo)

**Repository:** [AWS Deployment Documentation](https://github.com/sulemanrepo/AWS-deployment-documentation)
