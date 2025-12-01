![Screenshot](Project-002/Screenshot (17).png)



# WordPress Migration to AWS Lightsail using All-in-One WP Migration

> This guide walks you through creating an AWS Lightsail WordPress instance and importing a WordPress backup using the All-in-One WP Migration plugin.

---

## 📋 Table of Contents

- [Prerequisites](#prerequisites)
- [1. Create AWS Lightsail Instance](#1-create-aws-lightsail-instance)
- [2. Retrieve WordPress Credentials](#2-retrieve-wordpress-credentials)
- [3. Access WordPress Admin Panel](#3-access-wordpress-admin-panel)
- [4. Install All-in-One WP Migration Plugin](#4-install-all-in-one-wp-migration-plugin)
- [5. Increase Upload File Size Limit](#5-increase-upload-file-size-limit)
- [6. Import WordPress Backup](#6-import-wordpress-backup)
- [7. Verification](#7-verification)
- [Troubleshooting](#troubleshooting)

---

## Prerequisites

Before starting, ensure you have:

- ✅ AWS account with access to Lightsail
- ✅ WordPress backup file (`.wpress` format)
- ✅ All-in-One WP Migration Unlimited Extension plugin file (`.zip`)

---

## 1. Create AWS Lightsail Instance

### Step 1.1 – Navigate to AWS Lightsail

1. Log in to your [AWS Console](https://console.aws.amazon.com/)
2. Search for **Lightsail** service
3. Click **Create instance**

### Step 1.2 – Configure Instance

1. **Select platform**: Linux/Unix
2. **Select blueprint**: WordPress (under Apps + OS)
3. **Choose instance plan**: Select based on your requirements
   - Recommended: At least **$5/month** plan for better performance
4. **Name your instance**: Give it a meaningful name (e.g., `wordpress-migration`)
5. Click **Create instance**

⏳ Wait for the instance to be in **Running** state (usually takes 1-2 minutes)

---

## 2. Retrieve WordPress Credentials

### Step 2.1 – Connect to Instance

1. Click on your instance name
2. Click the **Connect using SSH** button (orange terminal icon)

### Step 2.2 – Get WordPress Password

In the SSH terminal, run:

```bash
cat /home/bitnami/bitnami_application_password
```

**Example output:**

```
your_generated_password_here
```

📝 **Copy this password** – you'll need it to log in to WordPress.

---

## 3. Access WordPress Admin Panel

### Step 3.1 – Get Instance Public IP

1. Go back to your Lightsail instance dashboard
2. Copy the **Public IP** address

### Step 3.2 – Login to WordPress

Open your browser and navigate to:

```
http://YOUR_INSTANCE_IP/wp-admin
```

**Example:**

```
http://54.123.45.67/wp-admin
```

**Login credentials:**
- **Username**: `user`
- **Password**: (paste the password from Step 2.2)

---

## 4. Install All-in-One WP Migration Plugin

### Step 4.1 – Navigate to Plugins

1. In WordPress dashboard, go to **Plugins** → **Add New**
2. Search for `All-in-One WP Migration`
3. Click **Install Now**
4. Click **Activate**

✅ The free version is now installed, but has a **512 MB upload limit**.

---

## 5. Increase Upload File Size Limit

> **Important:** The free version of All-in-One WP Migration has a maximum upload size of **512 MB**. To import larger backups, you need the **Unlimited Extension**.

### Step 5.1 – Upload Unlimited Extension Plugin

1. Go to **Plugins** → **Add New**
2. Click **Upload Plugin** (at the top)
3. Click **Choose File**
4. Select the `all-in-one-wp-migration-unlimited-extension.zip` file from your computer
5. Click **Install Now**
6. Click **Activate**

### Step 5.2 – Verify File Size Limit (Optional)

You can verify the increased limit by:

1. Going to **All-in-One WP Migration** → **Import**
2. The upload limit should now show **Unlimited** or **~100 GB**

> **Note:** If you don't have the unlimited extension, you can also increase PHP upload limits manually by modifying server configuration files, but using the extension is recommended.

---

## 6. Import WordPress Backup

### Step 6.1 – Start Import Process

1. In WordPress dashboard, navigate to:
   ```
   All-in-One WP Migration → Import
   ```

2. Click on **Import From** → **File**

3. Select your `.wpress` backup file

### Step 6.2 – Wait for Upload

- The plugin will upload and process your backup file
- ⏳ **Do not close the browser** during this process
- Progress bar will show upload and extraction status

### Step 6.3 – Complete Import

1. Once import is complete, you'll see a success message
2. Click **Finish** when prompted
3. You'll be automatically logged out

### Step 6.4 – Login with New Credentials

- Use the **original site's credentials** (from your backup)
- Not the temporary `user` account

---

## 7. Verification

### Step 7.1 – Check Your Site

Visit your site:

```
http://YOUR_INSTANCE_IP
```

### Step 7.2 – Verify Content

- ✅ Check pages and posts
- ✅ Verify images are loading
- ✅ Test navigation menus
- ✅ Check theme and plugins

### Step 7.3 – Update Permalinks

1. Go to **Settings** → **Permalinks**
2. Click **Save Changes** (even without making changes)
3. This regenerates the `.htaccess` file

---

## Troubleshooting

### Issue: Cannot access wp-admin

**Solution:**

```bash
# SSH into your instance and restart services
sudo /opt/bitnami/ctlscript.sh restart
```

### Issue: Upload size limit still showing 512 MB

**Solution:**
1. Verify the unlimited extension is activated
2. Try deactivating and reactivating both plugins
3. Clear browser cache

### Issue: White screen after import

**Solution:**

```bash
# SSH into instance
sudo /opt/bitnami/ctlscript.sh restart apache
sudo /opt/bitnami/ctlscript.sh restart mysql
```

### Issue: Images not loading

**Solution:**
1. Go to **Settings** → **Permalinks** → **Save Changes**
2. Go to **Settings** → **General**
3. Update WordPress Address (URL) and Site Address (URL) to your new IP

---

## 🎯 Next Steps

After successful migration:

1. **Attach a Static IP** (optional but recommended)
   - Prevents IP changes on instance restart
   - Go to Lightsail dashboard → Networking → Create static IP

2. **Configure Domain Name**
   - Point your domain to the Lightsail static IP
   - Update WordPress URLs in **Settings** → **General**

3. **Enable SSL/HTTPS**
   - Use Bitnami's built-in Let's Encrypt integration
   - Or use the Bitnami HTTPS configuration tool

4. **Set Up Backups**
   - Enable Lightsail automatic snapshots
   - Schedule regular All-in-One WP Migration exports

5. **Security Hardening**
   - Change default `user` account
   - Install security plugins (Wordfence, iThemes Security)
   - Update all plugins and themes

---

## 📚 Additional Resources

- [AWS Lightsail Documentation](https://docs.aws.amazon.com/lightsail/)
- [Bitnami WordPress Documentation](https://docs.bitnami.com/aws/apps/wordpress/)
- [All-in-One WP Migration Documentation](https://help.servmask.com/)

---

## 📝 License

This project is open source and available under the [MIT License](LICENSE).

## 🤝 Contributing

Contributions, issues, and feature requests are welcome!

---

**Author:** [Your Name](https://github.com/sulemanrepo)

**Repository:** [AWS Deployment Documentation](https://github.com/sulemanrepo/AWS-deployment-documentation)
