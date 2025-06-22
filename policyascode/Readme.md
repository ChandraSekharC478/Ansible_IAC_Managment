# 📘 S3 Versioning Policy as Code with Ansible

This README explains how to use Ansible to enforce **S3 bucket versioning** across all AWS S3 buckets in your account using the `amazon.aws` collection.

---

## 🎯 Objective

Automatically enforce **versioning** for all S3 buckets using an Ansible Playbook.

---

## 📦 Requirements

* **Ansible v2.16+** (Older versions may cause compatibility issues)
* **amazon.aws Collection**:

  ```bash
  ansible-galaxy collection install amazon.aws
  ```
* **AWS Credentials** properly configured via:

  * Environment variables (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`)
  * `~/.aws/credentials`
  * IAM role if running on EC2

---

## 📜 Playbook: `s3_versioning.yaml`

```yaml
---
- name: Enforce S3 bucket versioning for all buckets
  hosts: localhost
  gather_facts: false
  connection: local

  tasks:
    - name: List all the S3 buckets
      amazon.aws.s3_bucket_info:
      register: result

    - name: Debug raw result output (optional)
      debug:
        var: result

    - name: Show the list of buckets
      ansible.builtin.debug:
        msg: "{{ result.buckets }}"

    - name: Enable versioning for all buckets
      amazon.aws.s3_bucket:
        name: "{{ item.name }}"
        versioning: true
      loop: "{{ result.buckets }}"
```

---

## 🧪 How to Run It

```bash
ansible-playbook s3_versioning.yaml
```

If using vault credentials:

```bash
ansible-playbook s3_versioning.yaml --ask-vault-pass
```

---

## 🛠️ Common Errors & Fixes

### 🔐 Vault Decryption Error

**Error:** `Attempting to decrypt but no vault secrets found`

* **Fix:** Add `--ask-vault-pass` or decrypt file:

  ```bash
  ansible-vault decrypt vault.yml
  ```

### ⚠️ Collection Compatibility

**Error:** `amazon.aws does not support Ansible version 2.15.13`

* **Fix:** Upgrade Ansible:

  ```bash
  pip install --upgrade ansible
  ```

  Or install compatible collection:

  ```bash
  ansible-galaxy collection install amazon.aws:3.1.1
  ```

### 🧭 Inventory Warning

**Warning:** `No inventory was parsed`

* **Fix:** Add:

  ```yaml
  connection: local
  ```

  Or provide explicit inventory if needed.

---

## ✅ Output Example

```
TASK [Show the list of buckets]
ok: [localhost] => {
    "msg": [
        {
            "name": "my-bucket-1",
            "creation_date": "2021-01-10T12:00:00Z"
        },
        ...
    ]
}
```

---

## 🔚 Next Steps

* Add logic to skip buckets with versioning already enabled
* Generate a report of versioning status
* Store bucket names in a log file
* Integrate into CI/CD pipeline for compliance enforcement

---
