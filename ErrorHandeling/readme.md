# Ansible Error Handling and Docker Setup - Learning Log

This project demonstrates how to use Ansible for installing system-level packages such as OpenSSH and Docker, while gracefully handling failures across managed nodes.

---

## ✅ Project Objective

* Ensure `openssh` and `openssl` are up-to-date.
* Check if Docker is installed, and install it only if it's not present.
* Handle failures gracefully using `ignore_errors`.
* Simulate failure on one node to ensure fault tolerance across inventory.
* Track what happens when `ignore_errors: yes` is **not** used.

---

## 📦 Playbook Summary

```yaml
- hosts: all
  become: true

  tasks:
    - name: Make sure the packages openssh and open ssl are uptodate
      ansible.builtin.apt:
        name: "{{ item }}"
        state: latest
      loop:
        - openssh
        - openssl
      ignore_errors: yes

    - name: verify the docker is exists
      ansible.builtin.command: docker --version
      register: output
      ignore_errors: yes

    - name: debugging the where failures are occouring
      ansible.builtin.debug:
        var: output

    - name: installation of docker only if and only if docker is not present
      ansible.builtin.apt:
        name: docker.io
        state: present
        update_cache: yes
      when:
        output.failed == true
```

---

## 🧾 Error Handling Log

### 1. ❌ `openssh` Package Not Found

* **Error:** `No package matching 'openssh' is available`
* **Cause:** Ubuntu does not provide a single `openssh` package.
* **Fix:** Replace with `openssh-client` and `openssh-server`.
* **Workaround Used:** `ignore_errors: yes` to allow execution to continue.

### 2. ❌ Docker Command Not Found

* **Error:** `No such file or directory: b'docker'`
* **Cause:** Docker not installed.
* **Fix:**

  ```yaml
  when: output.failed == true
  ```

  to conditionally install Docker.

### 3. ❌ Docker.io Package Not Found

* **Error:** `No package matching 'docker.io' is available`
* **Cause:** Apt cache outdated or universe repo not enabled.
* **Fix:** Add `update_cache: yes` or ensure apt repositories are complete.

---

## ❌ What Happens Without `ignore_errors: yes`

### 1. Task: Install `openssh`

```yaml
- name: Install openssh and openssl
  apt:
    name: "{{ item }}"
    state: latest
  loop:
    - openssh
    - openssl
```

* **Error Output:**

  ```
  No package matching 'openssh' is available
  ```
* **Reason:** The playbook **fails immediately**, and other tasks are skipped because the default behavior is to stop execution on any failure.

### 2. Task: `docker --version` command

```yaml
- name: Check docker version
  command: docker --version
  register: output
```

* **Error Output:**

  ```
  [Errno 2] No such file or directory: b'docker'
  ```
* **Reason:** Playbook fails and halts since Docker isn’t installed.

### Solution:

Use `ignore_errors: yes` to continue execution even if a failure occurs.

---

## 💡 Key Learnings

* Use `ignore_errors: yes` for fault-tolerant automation.
* Register command output and use `.failed` or `.rc` for conditional tasks.
* Always run `update_cache: yes` before installing packages.
* Simulate errors on selected nodes using `inventory_hostname`.

---

## 🚀 Next Steps

* Use `block`, `rescue`, and `always` for more advanced error handling.
* Create reusable roles for package installation.
* Add notifications on failure using `ansible.builtin.mail` or Slack API.

---
## Author

***chandrasekhar***
