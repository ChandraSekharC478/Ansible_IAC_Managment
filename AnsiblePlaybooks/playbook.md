
---

## ✅ Objective

This guide walks through:

1. Enabling **passwordless SSH authentication** for an EC2 instance.
2. Creating an **inventory file** for Ansible.
3. Running your **first Ansible playbook** to install Apache and serve a basic HTML page.

---

## 🔐 Step 1: Enable Passwordless SSH Authentication

1. SSH into your EC2 VM using the `.pem` file:

   ```bash
   ssh -i "iam.pem" ubuntu@<EC2_PUBLIC_IP>
   ```

2. Edit the cloud-init SSH config to allow password authentication:

   ```bash
   sudo vim /etc/ssh/sshd_config.d/60-cloudimg-settings.conf
   ```

   Add or modify:

   ```
   PasswordAuthentication yes
   ```

3. Edit main SSH config (optional but good practice):

   ```bash
   sudo vim /etc/ssh/sshd_config
   ```

   Uncomment or add:

   ```
   PasswordAuthentication yes
   ```

4. Restart SSH service:

   ```bash
   sudo systemctl restart ssh
   ```

5. Set a password for the `ubuntu` user:

   ```bash
   sudo passwd ubuntu
   ```

6. From your local machine, copy your public SSH key:

   ```bash
   ssh-copy-id ubuntu@<EC2_PUBLIC_IP>
   ```

---

## 📁 Step 2: Create Your Ansible Inventory File

Create a file named `inventory.ini`:

```ini
[web]
<EC2_PUBLIC_IP> ansible_user=ubuntu
```

---

## 🧾 Step 3: Create Your First Playbook

Create a file named `webserver.yaml` with the following content:

```yaml
---
- hosts: all
  become: true
  tasks:
    - name: Install apache httpd
      ansible.builtin.apt:
        name: apache2
        state: present
        update_cache: yes

    - name: Copy file owner and permissions
      ansible.builtin.copy:
        src: index.html
        dest: /var/www/html/index.html
        owner: root
        group: root
        mode: '0644'
```

---

## 🌐 Step 4: Prepare the HTML File

Create a simple HTML file named `index.html` in the same directory:

```html
<!DOCTYPE html>
<html>
<head>
  <title>My First Ansible Web Page</title>
</head>
<body>
  <h1>Hello from Ansible!</h1>
</body>
</html>
```

---

## 🚀 Step 5: Run the Playbook

Execute the playbook using Ansible:

```bash
ansible-playbook -i inventory.ini webserver.yaml
```

---

## ✅ Result

* Apache server is installed.
* `index.html` is copied to `/var/www/html/`.
* Visiting `http://<EC2_PUBLIC_IP>` shows your custom HTML page.

---

