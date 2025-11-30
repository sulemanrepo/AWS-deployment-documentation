Deploy WordPress on EC2 (Ubuntu) with Domain & Let’s Encrypt SSL

This guide explains how to deploy WordPress on an Ubuntu EC2 instance, configure the LAMP stack, set up a database, attach a domain, and enable SSL using Let’s Encrypt.

📌 Table of Contents

Launch EC2 Instance

Connect to EC2

Install LAMP Stack

Install & Configure WordPress

Configure MySQL for WordPress

Configure wp-config.php

Fix Error Establishing DB Connection

Attach Domain to EC2

Install Let’s Encrypt SSL

Access WordPress

Final Notes

1. Launch EC2 Instance

Use the following configuration:

AMI: Ubuntu Server 22.04 LTS

Instance type: t2.micro (Free-tier eligible)

Security Group: Allow

HTTP (80)

HTTPS (443)

SSH (22)

Key Pair: Create a new one and download it

Launch the instance

2. Connect to EC2

Use SSH to access your instance:

ssh -i "Downloads/task1Key.pem" ubuntu@<EC2-Public-IP>


Example:

ssh -i "Downloads/task1Key.pem" ubuntu@35.181.48.139

3. Install LAMP Stack
✔ Update Packages
sudo apt update

✔ Install Apache
sudo apt install apache2 -y

✔ Install MySQL
sudo apt install mysql-server -y
sudo mysql_secure_installation

✔ Install PHP
sudo apt install php libapache2-mod-php php-mysql php-cli php-curl php-json php-cgi php-xml php-mbstring -y

4. Install & Configure WordPress
Download WordPress
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz

Move Files
sudo mv wordpress /var/www/html/
sudo chown -R www-data:www-data /var/www/html/wordpress
sudo chmod -R 755 /var/www/html/wordpress

5. Configure MySQL for WordPress
Login to MySQL:
sudo mysql -u root -p

Create Database & User:
CREATE DATABASE wordpress;
CREATE USER 'wordpressuser'@'localhost' IDENTIFIED BY 'password123';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpressuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;

6. Configure wp-config.php
cd /var/www/html/wordpress
sudo cp wp-config-sample.php wp-config.php
sudo nano wp-config.php


Update:

define( 'DB_NAME', 'wordpress' );
define( 'DB_USER', 'wordpressuser' );
define( 'DB_PASSWORD', 'password123' );
define( 'DB_HOST', 'localhost' );

7. Fix: Error Establishing a Database Connection
✔ Verify Config File
sudo nano /var/www/html/wordpress/wp-config.php

✔ Check Database

Inside MySQL:

SHOW DATABASES;


If missing:

CREATE DATABASE wordpress;

✔ Verify User
SELECT user, host FROM mysql.user;


If missing:

CREATE USER 'wordpressuser'@'localhost' IDENTIFIED BY 'password123';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpressuser'@'localhost';
FLUSH PRIVILEGES;

✔ Test DB Connectivity
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


Test in browser:

http://<EC2-IP>/wordpress/testdb.php


Delete after testing:

sudo rm /var/www/html/wordpress/testdb.php

✔ Restart Services
sudo systemctl restart apache2
sudo systemctl restart mysql
sudo tail -f /var/log/mysql/error.log

8. Attach Domain to EC2

Go to your domain registrar → Add DNS:

A Record
Host	Value	TTL
@	EC2 Public IP	300
www	EC2 Public IP	300

Propagation: 5–30 minutes

9. Install Let’s Encrypt SSL
Install Certbot
sudo apt update
sudo apt install certbot python3-certbot-apache -y

Generate SSL
sudo certbot --apache

Test Auto-Renew
sudo certbot renew --dry-run

10. Access Your WordPress Site

Default:

http://<your-domain>/wordpress


If moved to root:

http://<your-domain>


Complete setup wizard.

11. Final Notes

Keep server updated:

sudo apt update && sudo apt upgrade -y


Backup wp-config.php

Consider moving WordPress to root /var/www/html/

Use CloudFront / WAF for production deployments
