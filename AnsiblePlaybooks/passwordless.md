# Passwordless SSH Setup to EC2 Using PEM and ssh-copy-id

This document outlines the steps to enable password-based login temporarily on an EC2 instance and then configure passwordless SSH using `ssh-copy-id`.

---

## ✅ Steps

### 1. SSH into EC2 Using PEM File
```bash
ssh -i "iam.pem" ubuntu@<public-ip-or-dns>
```

---

### 2. Enable Password Authentication

#### a. Open and edit the cloud-init SSH config file:
```bash
sudo vim /etc/ssh/sshd_config.d/60-cloudimg-settings.conf
```
- Add or change this line:
```conf
PasswordAuthentication yes
```

#### b. Also edit the main SSH config file:
```bash
sudo vim /etc/ssh/sshd_config
```
- Uncomment or add:
```conf
PasswordAuthentication yes
```

---

### 3. Restart SSH Service
```bash
sudo systemctl restart ssh
```

---

### 4. Set a Password for the Ubuntu User
```bash
sudo passwd ubuntu
```
- Enter and confirm a new password.

---

### 5. Copy Public Key to EC2 for Passwordless Login
From your **local machine**:
```bash
ssh-copy-id ubuntu@<public-ip>
```
- Enter the password you set in step 4 when prompted.

---

### 6. Test Passwordless SSH
```bash
ssh ubuntu@<public-ip>
```
- You should now be able to log in without a password.

---

## 🔁 Summary
This method lets you temporarily enable password-based login to push your public SSH key. Once done, you can revert `PasswordAuthentication` back to `no` in both config files if desired, for security.

---

**Note:** This is a temporary measure and should be used with caution on production environments.

---

**Author:** Chandrasekhar
**Date:** June 2025