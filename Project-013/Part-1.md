# Ansible Playbook for Remote Package Installation & Updates

This document explains how to set up an Ansible control node (Master) and run playbooks on a remote managed node to install or update packages.  
The process includes generating SSH keys, allowing passwordless SSH authentication, configuring Ansible inventory, and executing playbooks.

---

## 📌 Step 1: Create EC2 Instances

Create **two EC2 Ubuntu servers** with the same security groups and configuration:

1. **Master Server**  
2. **Node Server**

Ensure SSH (port 22) is allowed.

---

## 📌 Step 2: Install Ansible on Master Server

SSH into the **Master Server**:

```bash
sudo apt update
sudo apt install -y ansible
```

### Generate SSH Keypair

```bash
ssh-keygen -t rsa -b 2048
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
```

---

## 📌 Step 3: Copy Master Public Key to Node Server

Display the public key:

```bash
cat ~/.ssh/id_rsa.pub
```

### On the Node Server

Create `.ssh` folder:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

Add the public key copied from Master:

```bash
echo "your-public-key" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

This enables passwordless SSH login.

---

## 📌 Step 4: Test SSH Connection From Master

From Master:

```bash
ssh ubuntu@node-instance-public-ip
```

If login succeeds, continue.

### Create the Ansible inventory file

```bash
sudo mkdir -p /etc/ansible
sudo nano /etc/ansible/hosts
```

Add the node’s **private IP**:

```
[nodes]
node-instance-private-ip
```

---

## 📌 Step 5: Create the Ansible Playbook (Install + Update Packages)

Create playbook:

```bash
nano install_update_packages.yml
```

Paste the content:

```yaml
---
- name: Install or update packages on node
  hosts: nodes
  become: yes
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes

    - name: Install specified packages
      apt:
        name: "{{ item }}"
        state: present
      loop:
        - curl
        - git
        - vim

    - name: Upgrade all packages
      apt:
        upgrade: dist
```

### Run the playbook

```bash
ansible-playbook install_update_packages.yml
```

---

## 📌 Step 6: Create a Second Playbook (Install Only cURL)

```bash
nano install_curl.yml
```

Paste:

```yaml
---
- name: Install curl on nodes
  hosts: nodes
  become: yes
  tasks:
    - name: Ensure curl is installed
      apt:
        name: curl
        state: present
```

Run:

```bash
ansible-playbook install_curl.yml
```

---

## 📌 Step 7: Validate the Installation

SSH into the Node again:

```bash
ssh ubuntu@node-instance-public-ip
```

Validate:

```bash
curl --version
```

If curl exists → your Ansible setup is working successfully.

---

## 🎉 **Complete!**

You have successfully:

✔ Installed Ansible  
✔ Set up passwordless SSH authentication  
✔ Configured Ansible inventory  
✔ Created and executed playbooks  
✔ Installed and updated packages on a remote node  

Your automation workflow is ready for expansion into larger infrastructure management.

