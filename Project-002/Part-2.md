# Manual WordPress Backup from AWS Lightsail Instance

> This guide demonstrates how to manually backup WordPress files and database from an AWS Lightsail instance and download them to your local environment.

---

## 📋 Table of Contents

- [Prerequisites](#prerequisites)
- [Overview](#overview)
- [Step 1: Navigate to WordPress Directory](#step-1-navigate-to-wordpress-directory)
- [Step 2: Compress WordPress Files](#step-2-compress-wordpress-files)
- [Step 3: Backup Database](#step-3-backup-database)
- [Step 4: Download Backups to Local Environment](#step-4-download-backups-to-local-environment)
- [Verification](#verification)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)

---

## Prerequisites

Before starting, ensure you have:

- ✅ Running AWS Lightsail WordPress instance
- ✅ SSH access to the instance
- ✅ Your private key file (`.pem`)
- ✅ Sufficient disk space on local machine
- ✅ Terminal/Command Prompt access

---

## Overview

This backup process creates two files:

1. **wordpress-backup.tar.gz** - All WordPress files (themes, plugins, uploads, etc.)
2. **wordpress-database-backup.sql** - Complete database dump

📦 **Total backup includes:**
- WordPress core files
- Themes and plugins
- Media uploads
- Database content
- Configuration files

---

## Step 1: Navigate to WordPress Directory

### 1.1 Connect to Your Instance

```bash
ssh -i /path/to/your-key.pem bitnami@YOUR_INSTANCE_IP
```

**Example:**

```bash
ssh -i Downloads/lightsail-key.pem bitnami@44.223.7.199
```

### 1.2 Go to WordPress Directory

```bash
cd /opt/bitnami/wordpress
```

### 1.3 Verify Location

```bash
pwd
ls -la
```

You should see WordPress files like `wp-config.php`, `wp-content/`, etc.

---

## Step 2: Compress WordPress Files

### 2.1 Create Compressed Backup

The WordPress directory can be large, so we'll compress it:

```bash
sudo tar -czvf wordpress-backup.tar.gz .
```

**Command breakdown:**
- `tar` - Archive utility
- `-c` - Create archive
- `-z` - Compress with gzip
- `-v` - Verbose (show progress)
- `-f` - Specify filename
- `.` - Current directory (all files)

⏳ This may take a few minutes depending on site size.

### 2.2 Move Backup to Home Directory

For easier access and download:

```bash
sudo mv wordpress-backup.tar.gz /home/bitnami/
```

### 2.3 Verify Backup File

```bash
ls -lh /home/bitnami/wordpress-backup.tar.gz
```

This will show the file size (e.g., `245M` for 245 megabytes).

---

## Step 3: Backup Database

### 3.1 Retrieve Database Credentials

First, view your WordPress configuration to get database credentials:

```bash
sudo cat /opt/bitnami/wordpress/wp-config.php
```

Look for these lines:

```php
define( 'DB_NAME', 'bitnami_wordpress' );
define( 'DB_USER', 'bn_wordpress' );
define( 'DB_PASSWORD', 'your_password_here' );
```

📝 **Note down:**
- `DB_NAME` (e.g., `bitnami_wordpress`)
- `DB_USER` (e.g., `bn_wordpress`)
- `DB_PASSWORD` (e.g., `your_password`)

### 3.2 Export Database

#### Option A: Using mysqldump (Traditional Method)

```bash
mysqldump -u DB_USER -pDB_PASSWORD DB_NAME > wordpress-database-backup.sql
```

**Example:**

```bash
mysqldump -u bn_wordpress -pYourPassword123 bitnami_wordpress > wordpress-database-backup.sql
```

> ⚠️ **Important:** No space between `-p` and password!

#### Option B: Using mariadb-dump (Recommended for Newer Instances)

If you get a deprecation warning or `mysqldump` is not found, use `mariadb-dump`:

```bash
/opt/bitnami/mariadb/bin/mariadb-dump -u DB_USER -pDB_PASSWORD DB_NAME > wordpress-database-backup.sql
```

**Example:**

```bash
/opt/bitnami/mariadb/bin/mariadb-dump -u bn_wordpress -pYourPassword123 bitnami_wordpress > wordpress-database-backup.sql
```

### 3.3 Move Database Backup

```bash
sudo mv wordpress-database-backup.sql /home/bitnami/
```

### 3.4 Verify Database Backup

```bash
ls -lh /home/bitnami/wordpress-database-backup.sql
```

You should see the SQL file with its size.

### 3.5 Set Proper Permissions

```bash
sudo chown bitnami:bitnami /home/bitnami/wordpress-backup.tar.gz
sudo chown bitnami:bitnami /home/bitnami/wordpress-database-backup.sql
```

---

## Step 4: Download Backups to Local Environment

### 4.1 Open New Terminal (Local Machine)

**Don't close your SSH session!** Open a new terminal/command prompt on your **local machine**.

### 4.2 Download WordPress Files

```bash
scp -i /path/to/your-key.pem bitnami@YOUR_INSTANCE_IP:/home/bitnami/wordpress-backup.tar.gz /local/directory
```

**Example for macOS/Linux:**

```bash
scp -i ~/Downloads/key123.pem bitnami@44.223.7.199:/home/bitnami/wordpress-backup.tar.gz ~/Desktop/backups/
```

**Example for Windows (PowerShell):**

```powershell
scp -i C:\Users\YourName\Downloads\key123.pem bitnami@44.223.7.199:/home/bitnami/wordpress-backup.tar.gz C:\Backups\
```

### 4.3 Download Database Backup

```bash
scp -i /path/to/your-key.pem bitnami@YOUR_INSTANCE_IP:/home/bitnami/wordpress-database-backup.sql /local/directory
```

**Example for macOS/Linux:**

```bash
scp -i ~/Downloads/key123.pem bitnami@44.223.7.199:/home/bitnami/wordpress-database-backup.sql ~/Desktop/backups/
```

**Example for Windows (PowerShell):**

```powershell
scp -i C:\Users\YourName\Downloads\key123.pem bitnami@44.223.7.199:/home/bitnami/wordpress-database-backup.sql C:\Backups\
```

⏳ **Download time depends on:**
- File size
- Internet connection speed
- Server location

---

## Verification

### On Local Machine

After downloads complete, verify the files:

#### macOS/Linux:

```bash
cd ~/Desktop/backups/
ls -lh
```

#### Windows (PowerShell):

```powershell
cd C:\Backups\
dir
```

### Check File Integrity

You should see two files:
- ✅ `wordpress-backup.tar.gz` (hundreds of MB typically)
- ✅ `wordpress-database-backup.sql` (size varies based on content)

### Test Archive Extraction (Optional)

```bash
# Create test directory
mkdir test-restore
cd test-restore

# Extract (don't run this in production!)
tar -xzvf ../wordpress-backup.tar.gz
```

---

## Troubleshooting

### Issue: Permission Denied Error

**Solution:**

```bash
# On Lightsail instance
sudo chmod 644 /home/bitnami/wordpress-backup.tar.gz
sudo chmod 644 /home/bitnami/wordpress-database-backup.sql
sudo chown bitnami:bitnami /home/bitnami/*.tar.gz
sudo chown bitnami:bitnami /home/bitnami/*.sql
```

### Issue: mysqldump Command Not Found

**Error message:**

```
bash: mysqldump: command not found
```

**Solution:**

Use the full path to mariadb-dump instead:

```bash
/opt/bitnami/mariadb/bin/mariadb-dump -u bn_wordpress -pYourPassword bitnami_wordpress > wordpress-database-backup.sql
```

### Issue: mysqldump Deprecated Warning

**Warning message:**

```
mysqldump is deprecated and will be removed in a future version. Use mariadb-dump instead.
```

**Solution:**

This is just a warning. You can continue using `mysqldump`, or switch to `mariadb-dump`:

```bash
/opt/bitnami/mariadb/bin/mariadb-dump -u DB_USER -pDB_PASSWORD DB_NAME > backup.sql
```

### Issue: "No space left on device"

**Solution:**

```bash
# Check disk space
df -h

# Remove old backups or unnecessary files
sudo rm /home/bitnami/old-backup.tar.gz

# Or create backup on a different volume
```

### Issue: SCP Connection Timeout

**Solution:**

```bash
# Verify instance is running
# Check security group allows SSH (port 22)
# Verify key file permissions
chmod 400 /path/to/key.pem
```

### Issue: Database Export Is Empty

**Solution:**

```bash
# Verify database credentials
sudo cat /opt/bitnami/wordpress/wp-config.php

# Test database connection
mysql -u DB_USER -pDB_PASSWORD DB_NAME -e "SHOW TABLES;"

# Check if database has data
mysql -u DB_USER -pDB_PASSWORD DB_NAME -e "SELECT COUNT(*) FROM wp_posts;"
```

---

## Best Practices

### 🔐 Security

- ✅ **Store backups securely** - Encrypt sensitive backups
- ✅ **Don't commit to Git** - Add `*.sql` and `*.tar.gz` to `.gitignore`
- ✅ **Remove from server** - Delete backups from Lightsail after downloading

```bash
# After successful download, remove from server
sudo rm /home/bitnami/wordpress-backup.tar.gz
sudo rm /home/bitnami/wordpress-database-backup.sql
```

### 📅 Backup Schedule

Create a backup routine:

- **Daily**: Database backups (smaller size)
- **Weekly**: Full WordPress files + database
- **Before updates**: Always backup before major changes

### 🏷️ Naming Convention

Use timestamps in filenames:

```bash
# Better naming
sudo tar -czvf wordpress-backup-$(date +%Y%m%d-%H%M%S).tar.gz .

# Example output: wordpress-backup-20241201-143025.tar.gz
```

### 💾 Storage Options

Consider storing backups in:

1. **Local external drive** - Offline security
2. **Cloud storage** - AWS S3, Google Drive, Dropbox
3. **Multiple locations** - Follow 3-2-1 backup rule

### 🔄 Automated Backups

For production sites, consider:

- AWS Lightsail automatic snapshots
- Cron jobs for automated backups
- Backup plugins (UpdraftPlus, BackupBuddy)

---

## 📚 Additional Resources

- [AWS Lightsail Snapshots](https://docs.aws.amazon.com/lightsail/latest/userguide/understanding-instance-snapshots-in-amazon-lightsail.html)
- [WordPress Backup Guide](https://wordpress.org/support/article/wordpress-backups/)
- [MySQL Backup Documentation](https://dev.mysql.com/doc/refman/8.0/en/backup-methods.html)

---

## 🎯 Next Steps

After backing up your WordPress site:

1. **Test restoration** on a local environment
2. **Set up automated backups** for regular intervals
3. **Store backups offsite** in cloud storage
4. **Document your backup process** for team members
5. **Create restoration procedure** documentation

---

## 📝 License

This project is open source and available under the [MIT License](LICENSE).

## 🤝 Contributing

Contributions, issues, and feature requests are welcome!

---

**Author:** [Your Name](https://github.com/sulemanrepo)

**Repository:** [AWS Deployment Documentation](https://github.com/sulemanrepo/AWS-deployment-documentation)
