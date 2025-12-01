# Deploy WordPress on EC2 (Ubuntu) with Domain & Let's Encrypt SSL

> This documentation explains the complete process of deploying WordPress on an Ubuntu EC2 instance, configuring LAMP stack, setting up the WordPress database, attaching a domain, and issuing a free Let's Encrypt SSL certificate.

---

## 📋 Table of Contents

- [1. Launch EC2 Instance](#1-launch-ec2-instance)
- [2. Connect to EC2 Instance](#2-connect-to-ec2-instance)
- [3. Install LAMP Stack](#3-install-lamp-stack)
- [4. Install & Configure WordPress](#4-install--configure-wordpress)
- [5. Configure MySQL Database](#5-configure-mysql-database-for-wordpress)
- [6. Configure WordPress](#6-configure-wordpress-wp-configphp)
- [7. Troubleshooting Database Connection](#7-fix-error-establishing-a-database-connection)
- [8. Attach Domain to EC2](#8-attach-domain-to-ec2-instance)
- [9. Install Let's Encrypt SSL](#9-install-lets-encrypt-ssl-https)
- [10. Access WordPress Site](#10-access-your-wordpress-site)
- [11. Final Notes](#11-final-notes)

---

## 1. Launch EC2 Instance

### Step 1 – Create EC2 Instance

1. **Choose AMI**: Ubuntu Server 22.04 LTS
2. **Instance type**: `t2.micro` (Free-tier eligible)
3. **Configure security group** and allow:
   - HTTP (Port 80)
   - HTTPS (Port 443)
   - SSH (Port 22)
4. **Create a new key pair** and download it
5. **Launch the instance**

---

## 2. Connect to EC2 Instance

Open your terminal and connect:

```bash
ssh -i "Downloads/task1Key.pem" ubuntu@<EC2-Public-IP>
```

**Example:**

```bash
ssh -i "Downloads/task1Key.pem" ubuntu@35.181.48.139
```

---

## 3. Install LAMP Stack (Linux, Apache, MySQL, PHP)

### Step 3.1 – Update packages

```bash
sudo apt update
```

### Step 3.2 – Install Apache

```bash
sudo apt install apache2 -y
```

### Step 3.3 – Install MySQL

```bash
sudo apt install mysql-server -y
sudo mysql_secure_installation
```

### Step 3.4 – Install PHP

```bash
sudo apt install php libapache2-mod-php php-mysql php-cli php-curl php-json php-cgi php-xml php-mbstring -y
```

---

## 4. Install & Configure WordPress

### Step 4.1 – Download WordPress

```bash
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
```

### Step 4.2 – Move WordPress files

```bash
sudo mv wordpress /var/www/html/
sudo chown -R www-data:www-data /var/www/html/wordpress
sudo chmod -R 755 /var/www/html/wordpress
```

---

## 5. Configure MySQL Database for WordPress

### Step 5.1 – Log into MySQL

```bash
sudo mysql -u root -p
```

Enter your password when prompted.

### Step 5.2 – Create Database & User

```sql
CREATE DATABASE wordpress;
CREATE USER 'wordpressuser'@'localhost' IDENTIFIED BY 'password123';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpressuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

---

## 6. Configure WordPress wp-config.php

```bash
cd /var/www/html/wordpress
sudo cp wp-config-sample.php wp-config.php
sudo nano wp-config.php
```

**Update these lines:**

```php
define( 'DB_NAME', 'wordpress' );
define( 'DB_USER', 'wordpressuser' );
define( 'DB_PASSWORD', 'password123' );
define( 'DB_HOST', 'localhost' );
```

Save and exit (`Ctrl + X`, then `Y`, then `Enter`).

---

## 7. Fix: Error Establishing a Database Connection

If you see the error: **"Error establishing a database connection"**, verify the following:

### 7.1 Check database configuration

```bash
sudo nano /var/www/html/wordpress/wp-config.php
```

Ensure correct values:
- `DB_NAME` = `wordpress`
- `DB_USER` = `wordpressuser`
- `DB_PASSWORD` = `password123`
- `DB_HOST` = `localhost`

### 7.2 Verify database exists

```sql
SHOW DATABASES;
```

If missing, create it:

```sql
CREATE DATABASE wordpress;
```

### 7.3 Verify MySQL user & permissions

```sql
SELECT user, host FROM mysql.user;
```

If missing, create user:

```sql
CREATE USER 'wordpressuser'@'localhost' IDENTIFIED BY 'password123';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpressuser'@'localhost';
FLUSH PRIVILEGES;
```

### 7.4 Test database connectivity

Create test file:

```bash
sudo nano /var/www/html/wordpress/testdb.php
```

Paste the following:

```php
<?php
$servername = "localhost";
$username = "wordpressuser";
$password = "password123";
$dbname = "wordpress";

$conn = new mysqli($servername, $username, $password, $dbname);

if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}
echo "Connected successfully";
?>
```

Open in browser:

```
http://<EC2-IP>/wordpress/testdb.php
```

If successful, **delete the file**:

```bash
sudo rm /var/www/html/wordpress/testdb.php
```

### 7.5 Restart Services

```bash
sudo systemctl restart apache2
sudo systemctl restart mysql
sudo tail -f /var/log/mysql/error.log
```

---

## 8. Attach Domain to EC2 Instance

### Step 8.1 – Go to your Domain Provider

(GoDaddy, Namecheap, WordPress.com, etc.)

Add DNS records:

| Type | Host | Value | TTL |
|------|------|-------|-----|
| A | @ | `<EC2-Public-IP>` | 300 |
| A (Optional) | www | `<EC2-Public-IP>` | 300 |

> **Note:** DNS propagation may take 5–30 minutes.

---

## 9. Install Let's Encrypt SSL (HTTPS)

### Step 9.1 Install Certbot

```bash
sudo apt update
sudo apt install certbot python3-certbot-apache -y
```

### Step 9.2 Generate SSL Certificate

```bash
sudo certbot --apache
```

**Follow the instructions:**
1. Enter domain name
2. Agree to terms
3. Redirect HTTP → HTTPS (recommended)

Certbot will automatically:
- ✔ Configure HTTPS
- ✔ Update Apache configuration
- ✔ Install certificates

### Step 9.3 Verify Auto-Renewal

```bash
sudo certbot renew --dry-run
```

---

## 10. Access Your WordPress Site

**WordPress Installation URL:**

```
http://<your-domain>/wordpress
```

Or if moved to root:

```
http://<your-domain>
```

Complete the on-screen setup:
- Site Name
- Admin Username
- Admin Password
- Email

---

## 11. Final Notes

### Keep your system updated

```bash
sudo apt update && sudo apt upgrade -y
```

### Important Recommendations

- ✅ Backup `wp-config.php` regularly
- ✅ Consider moving WordPress to `/var/www/html/` root
- ✅ Use CloudFront or WAF for production environments
- ✅ Set up regular backups
- ✅ Keep WordPress and plugins updated

---

## 📝 License

This project is open source and available under the [MIT License](LICENSE).

## 🤝 Contributing

Contributions, issues, and feature requests are welcome!

---

**Author:** [Your Name](https://github.com/sulemanrepo)

**Repository:** [AWS Deployment Documentation](https://github.com/sulemanrepo/AWS-deployment-documentation)
