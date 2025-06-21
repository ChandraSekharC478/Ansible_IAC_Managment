
````markdown
# 🚀 Ansible-Based EC2 Automation Project

This project demonstrates how to use **Ansible** to automate AWS EC2 instance provisioning, secure passwordless SSH setup, and OS-specific instance shutdowns.

---

## 📌 Project Objectives

1. **Provision 3 EC2 Instances** in the same region:
   - 2 Ubuntu instances
   - 1 Amazon Linux instance
   - Use of **loops** to avoid redundancy
2. **Set up Passwordless SSH**:
   - Public key-based authentication
   - Manual fallback via password-based access
3. **Stop Ubuntu Instances** selectively via Ansible

---

## 🧰 Prerequisites

- AWS CLI configured with credentials
- `boto3` and `botocore` libraries installed
- `ansible` installed
- IAM user with EC2 access
- SSH key pair (.pem) already created in AWS
- Security group with port `22` open

### Install Requirements

```bash
pip install boto3
ansible-galaxy collection install amazon.aws
````

---

## 🔐 Ansible Vault for Secure Credentials

To avoid hardcoding secrets:

### 1. Create Vault Password File

```bash
openssl rand -base64 2048 > vault.pass
```

### 2. Create Encrypted Credentials File

```bash
ansible-vault create group_vars/all/pass.yml --vault-password-file vault.pass
```

Add the following to `pass.yml`:

```yaml
ec2_access_key: YOUR_AWS_ACCESS_KEY
ec2_secret_key: YOUR_AWS_SECRET_KEY
```

---

## 📦 Task 1: Provision EC2 Instances (`ec2_provision.yml`)

```yaml
- name: Launch EC2 Instance
  hosts: localhost
  connection: local
  vars_files:
    - group_vars/all/pass.yml
  tasks:
    - name: Start EC2 instances
      amazon.aws.ec2_instance:
        name: "{{ item.name }}"
        key_name: "iam"
        instance_type: "t2.micro"
        security_group: default
        region: us-west-1
        aws_access_key: "{{ ec2_access_key }}"
        aws_secret_key: "{{ ec2_secret_key }}"
        network:
          assign_public_ip: true
        image_id: "{{ item.image }}"
        tags:
          Environment: Testing
      loop:
        - {image: "ami-0cbad6815f3a09a6d", name: "ansiblenode1"}     # Amazon Linux
        - {image: "ami-014e30c8a36252ae5", name: "ansiblenode2"}     # Ubuntu
        - {image: "ami-014e30c8a36252ae5", name: "ansiblenode3"}     # Ubuntu
```

### Run

```bash
ansible-playbook ec2_provision.yml --vault-password-file vault.pass
```

---

## 🔐 Task 2: Passwordless SSH Authentication

### ✅ Option 1: Using Public Key (Recommended)

```bash
ssh-copy-id -f "-o IdentityFile <PATH TO PEM FILE>" ubuntu@<INSTANCE-PUBLIC-IP>
```

* `-f`: Force overwrite existing key
* `-o IdentityFile`: Specifies the `.pem` file used to connect
* `ubuntu@<IP>`: Login user and public IP

Repeat for all instances.

---

### 🔄 Option 2: Manual Password Authentication (if needed)

1. **Edit the config file**:

```bash
sudo nano /etc/ssh/sshd_config.d/60-cloudimg-settings.conf
```

2. **Change this line**:

```
PasswordAuthentication yes
```

3. **Restart SSH**:

```bash
sudo systemctl restart ssh
```

> After that, you can manually copy the key or proceed with `ssh-copy-id`.

---

## 🛑 Task 3: Shutdown Ubuntu Instances

### Playbook: `shutdown_ubuntu.yml`

```yaml
- name: Shutdown Ubuntu Hosts
  hosts: all
  become: true
  tasks:
    - name: Shutdown Ubuntu machines
      ansible.builtin.command: /sbin/shutdown -t now
      when: ansible_facts['os_family'] == 'Debian'
```

> Ubuntu instances are under the `Debian` OS family; Amazon Linux falls under `RedHat`.

---

## 📁 Folder Structure

```
ansible-ec2-project/
├── ec2_provision.yml
├── shutdown_ubuntu.yml
├── vault.pass
├── group_vars/
│   └── all/
│       └── pass.yml
├── inventory/
│   └── hosts.ini
```

---

## ✅ Run Summary

```bash
# Step 1: Launch EC2s
ansible-playbook ec2_provision.yml --vault-password-file vault.pass

# Step 2: Set up Passwordless SSH (manual or with ssh-copy-id)

# Step 3: Shutdown Ubuntu Machines
ansible-playbook -i inventory/hosts.ini shutdown_ubuntu.yml
```

---

## 👨‍💻 Author

**Chandra Sekhar**
DevOps Engineer | AWS | Ansible | Terraform | Jenkins
📧 [chandrasekarc784@gmail.com](mailto:your.email@example.com)
🔗 [www.linkedin.com/in/chandrasekhar-c](#)

---

## 📝 Notes

* Replace AMI IDs with latest verified ones for your region.
* Make sure `.pem` file permissions are `chmod 400`.
* Use Ansible dynamic inventory for scaling this further.

