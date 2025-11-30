Deploy WordPress on EC2 (Ubuntu) with Domain & Let’s Encrypt SSL

This documentation explains the complete process of deploying WordPress on an Ubuntu EC2 instance, configuring LAMP stack, setting up the WordPress database, attaching a domain, and issuing a free Let’s Encrypt SSL certificate.

1. Launch EC2 Instance
Step 1 – Create EC2 Instance

Choose AMI: Ubuntu Server 22.04 LTS

Instance type: t2.micro (Free-tier eligible)

Configure security group and allow:

HTTP (80)

HTTPS (443)

SSH (22)

Create a new key pair and download it.

Launch the instance.

2. Connect to EC2 Instance

Open your terminal and connect:

ssh -i "Downloads/task1Key.pem" ubuntu@<EC2-Public-IP>


Example:

ssh -i "Downloads/task1Key.pem" ubuntu@35.181.48.139

3. Install LAMP Stack (Linux, Apache, MySQL, PHP)
Step 3.1 – Update packages
sudo apt update

Step 3.2 – Install Apache
sudo apt install apache2 -y

Step 3.3 – Install MySQL
sudo apt install mysql-server -y
sudo mysql_secure_installation

Step 3.4 – Install PHP
sudo apt install php libapache2-mod-php php-mysql php-cli php-curl php-json php-cgi php-xml php-mbstring -y

4. Install & Configure WordPress
Step 4.1 – Download WordPress
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz

Step 4.2 – Move WordPress files
sudo mv wordpress /var/www/html/
sudo chown -R www-data:www-data /var/www/html/wordpress
sudo chmod -R 755 /var/www/html/wordpress

5. Configure MySQL Database for WordPress
Step 5.1 – Log into MySQL
sudo mysql -u root -p


Enter your password.

Step 5.2 – Create Database & User
CREATE DATABASE wordpress;
CREATE USER 'wordpressuser'@'localhost' IDENTIFIED BY 'password123';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpressuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;

6. Configure WordPress wp-config.php
cd /var/www/html/wordpress
sudo cp wp-config-sample.php wp-config.php
sudo nano wp-config.php


Update these lines:

define( 'DB_NAME', 'wordpress' );
define( 'DB_USER', 'wordpressuser' );
define( 'DB_PASSWORD', 'password123' );
define( 'DB_HOST', 'localhost' );


Save and exit.

7. Fix: Error Establishing a Database Connection

If you see the error: “Error establishing a database connection”, verify:

7.1 Check database configuration
sudo nano /var/www/html/wordpress/wp-config.php


Ensure correct:

DB_NAME = wordpress
DB_USER = wordpressuser
DB_PASSWORD = password123
DB_HOST = localhost

7.2 Verify database exists
SHOW DATABASES;


If missing:

CREATE DATABASE wordpress;

7.3 Verify MySQL user & permissions
SELECT user, host FROM mysql.user;


If missing:

CREATE USER 'wordpressuser'@'localhost' IDENTIFIED BY 'password123';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpressuser'@'localhost';
FLUSH PRIVILEGES;

7.4 Test database connectivity

Create test file:

sudo nano /var/www/html/wordpress/testdb.php


Paste:

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


Open:

http://<EC2-IP>/wordpress/testdb.php


If successful, delete the file:

sudo rm /var/www/html/wordpress/testdb.php

7.5 Restart Services
sudo systemctl restart apache2
sudo systemctl restart mysql
sudo tail -f /var/log/mysql/error.log

8. Attach Domain to EC2 Instance
Step 8.1 – Go to your Domain Provider (GoDaddy, Namecheap, WordPress.com, etc.)

Add DNS records:

A Record

Host: @

Value: <EC2-Public-IP>

TTL: 300

Optional:

www → same IP

Propagation may take 5–30 minutes.

9. Install Let’s Encrypt SSL (HTTPS)
Step 9.1 Install Certbot
sudo apt update
sudo apt install certbot python3-certbot-apache -y

Step 9.2 Generate SSL Certificate
sudo certbot --apache


Follow instructions:

Enter domain name

Agree to terms

Redirect HTTP → HTTPS (recommended)

Certbot will automatically:
✔ Configure HTTPS
✔ Update Apache configuration
✔ Install certificates

Step 9.3 Verify Auto-Renewal
sudo certbot renew --dry-run

10. Access Your WordPress Site
WordPress Installation URL:
http://<your-domain>/wordpress


or if moved to root:

http://<your-domain>


Complete the on-screen setup:

Site Name

Admin Username

Admin Password

Email

11. Final Notes

Keep your system updated:

sudo apt update && sudo apt upgrade -y


Backup wp-config.php

Consider moving WordPress to /var/www/html/ root

Use CloudFront or WAF for production
