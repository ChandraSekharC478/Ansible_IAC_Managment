Absolutely! Here's a comprehensive `README.md` that walks through all the steps—from installing Ansible to setting up password-less SSH access using a PEM key.

---

### 📘 `README.md`: Ansible Setup with Passwordless SSH on Ubuntu

````markdown
# Ansible Setup with Passwordless SSH Access

This guide walks you through the steps to:
- Install Ansible on a control node (e.g., your local Mac or Ubuntu machine)
- Configure SSH key-based authentication using a `.pem` key (commonly from AWS EC2)
- Connect to a remote Ubuntu server without needing a password
- Test the setup and prepare for Ansible usage

---

## 🧰 Prerequisites

- A local machine with macOS or Linux
- Python 3 installed (Ansible dependency)
- A `.pem` SSH private key (e.g., `ansible.pem`)
- Access to a remote Ubuntu server (e.g., AWS EC2)
- Internet connection for package downloads

---

## 🚀 Step 1: Install Ansible (on the local machine)

### For macOS:
```bash
brew install ansible
````

### For Ubuntu:

```bash
sudo apt update
sudo apt install -y software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install -y ansible
```

To verify:

```bash
ansible --version
```

---

## 🔑 Step 2: Set Correct Permissions on the PEM File

Ensure your `.pem` file has restricted permissions:

```bash
chmod 600 ~/Downloads/ansible.pem
```

---

## 📄 Step 3: Generate Public Key from PEM File

```bash
ssh-keygen -y -f ~/Downloads/ansible.pem > ~/Downloads/ansible.pem.pub
```

This creates the corresponding public key required for SSH authorization.

---

## 📤 Step 4: Copy Public Key to Remote Server

### Create `.ssh` directory locally if missing:

```bash
mkdir -p ~/.ssh
```

### Use `ssh-copy-id` to upload the public key:

```bash
ssh-copy-id -i ~/Downloads/ansible.pem.pub ubuntu@<REMOTE_PUBLIC_IP>
```

> Replace `<REMOTE_PUBLIC_IP>` with your actual server IP (e.g., `13.221.127.250`).

If `ssh-copy-id` fails, use manual copy:

```bash
cat ~/Downloads/ansible.pem.pub | ssh -i ~/Downloads/ansible.pem ubuntu@<REMOTE_PUBLIC_IP> \
"mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

---

## 🔐 Step 5: Connect to Remote Server (Verify SSH Works)

Test the connection:

```bash
ssh -i ~/Downloads/ansible.pem ubuntu@<REMOTE_PUBLIC_IP>
```

You should log in without a password.

---

## 📁 Step 6: Optional - Create SSH Config File (Simplify SSH)

Edit or create `~/.ssh/config`:

```bash
nano ~/.ssh/config
```

Add:

```
Host my-server
    HostName <REMOTE_PUBLIC_IP>
    User ubuntu
    IdentityFile ~/Downloads/ansible.pem
```

Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`).

Now connect with just:

```bash
ssh my-server
```

---

## ✅ Final Verification

Try using Ansible's `ping` module:

```bash
ansible all -i <REMOTE_PUBLIC_IP>, -u ubuntu --private-key=~/Downloads/ansible.pem -m ping
```

Expected output:

```json
<REMOTE_PUBLIC_IP> | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

---

## 📝 Notes

* Ensure `python3` is installed on the remote machine (Ansible requires it).
* The PEM key should only be readable by your user (`chmod 600` is critical).
* Ansible does not work with password-based SSH unless explicitly configured.

---

## 🔗 Resources

* [Ansible Docs](https://docs.ansible.com/)
* [SSH Key Authentication Guide](https://www.ssh.com/academy/ssh/key)


## created by
  chandrasekhar