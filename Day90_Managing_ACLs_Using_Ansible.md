# 🚀 Day 90: Managing ACLs Using Ansible  
Advanced Permission Management Across App Servers

---

<img width="751" height="768" alt="Screenshot 2025-11-28 105613" src="https://github.com/user-attachments/assets/4ec509e4-c9ea-4d42-95e2-1cb1bfd6e762" />
<br>

---

## 📌 **Task Overview**

The Nautilus DevOps team requires secure file-level access management across all application servers within the Stratos Datacenter.  
To achieve this, they want to:

✔ Create specific files on each app server  
✔ Apply **fine-grained ACLs** using Ansible  
✔ Assign specific users/groups custom permissions  
✔ Automate everything — **no manual edits**

ACLs (Access Control Lists) allow more granular permission management beyond the traditional *owner–group–others* model.

Your job:

### 🔐 **ACL Requirements**
| App Server | File Path | Entity | Entity Type | Permissions |
|------------|-----------|--------|--------------|-------------|
| stapp01 | `/opt/security/blog.txt` | tony | group | `r` |
| stapp02 | `/opt/security/story.txt` | steve | user | `rw` |
| stapp03 | `/opt/security/media.txt` | banner | group | `rw` |

All tasks must be automated using **Ansible** via a single playbook:  📄 `/home/thor/ansible/playbook.yml`  

---

## 🛠 **Prerequisites**

### 1️⃣ Generate SSH Keys on Jump Host
```bash
ssh-keygen
ssh-copy-id tony@stapp01
ssh-copy-id steve@stapp02
ssh-copy-id banner@stapp03
```
### 2️⃣ Configure Sudo Access

Login to each app server and add respective users to sudoers to allow required operations (file creation, ACL modification).  

## 📁 Inventory File

The inventory already exists at: `/home/thor/ansible/inventory`

It should contain entries similar to: (either with a password or ssh-key)
```ini
stapp01 ansible_host=172.16.238.10 ansible_user=tony ansible_ssh_private_key_file=~/.ssh/id_rsa ansible_ssh_common_args='-o StrictHostKeyChecking=no'
stapp02 ansible_host=172.16.238.11 ansible_user=steve ansible_ssh_private_key_file=~/.ssh/id_rsa ansible_ssh_common_args='-o StrictHostKeyChecking=no'
stapp03 ansible_host=172.16.238.12 ansible_user=banner ansible_ssh_private_key_file=~/.ssh/id_rsa ansible_ssh_common_args='-o StrictHostKeyChecking=no
```

## 📜 Ansible Playbook - playbook.yml

Create the file:

📄 `/home/thor/ansible/playbook.yml`

```yml
---
- hosts: stapp01
  become: yes
  tasks:
    - name: Create empty file blog.txt
      ansible.builtin.file:
        path: /opt/security/blog.txt
        state: touch

    - name: Set ACL for group tony (read only)
      ansible.posix.acl:
        path: /opt/security/blog.txt
        entity: tony
        etype: group
        permissions: r
        state: present

- hosts: stapp02
  become: yes
  tasks:
    - name: Create empty file story.txt
      ansible.builtin.file:
        path: /opt/security/story.txt
        state: touch

    - name: Set ACL for user steve (read+write)
      ansible.posix.acl:
        path: /opt/security/story.txt
        entity: steve
        etype: user
        permissions: rw
        state: present

- hosts: stapp03
  become: yes
  tasks:
    - name: Create empty file media.txt
      ansible.builtin.file:
        path: /opt/security/media.txt
        state: touch

    - name: Set ACL for group banner (read+write)
      ansible.posix.acl:
        path: /opt/security/media.txt
        entity: banner
        etype: group
        permissions: rw
        state: present
```

### 🔎 Explanation
- `file module` with state: touch → creates an empty file if it doesn’t exist.
- `acl module` (from ansible.posix collection) → manages Access Control Lists.
    - `entity` → the user or group name.
    - `etype` → user or group.
    - `permissions` → r, rw, etc.
    - `state`: present → ensures the ACL entry exists.
---

## 📡 Connectivity Check  

```bash
ansible -i /home/thor/ansible/inventory all -m ping
```

Expected output: 

<img width="605" height="438" alt="Screenshot 2025-11-28 110215" src="https://github.com/user-attachments/assets/ec44bd21-448d-431f-b399-6b5d8bc8a23e" />
<br>

---

## ▶️ Run the Playbook
```bash
ansible-playbook -i /home/thor/ansible/inventory /home/thor/ansible/playbook.yml
```

<img width="933" height="774" alt="Screenshot 2025-11-28 110715" src="https://github.com/user-attachments/assets/57fee043-6c3d-414c-88f0-41d4ebfdc715" />
<br>

---

## 🔍 Verification
### ✔ Check file presence, permissions & ACLs
```bash
thor@jumphost ~$ ssh tony@stapp01 ls -l /opt/security/
total 0
-rw-r--r--+ 1 root root 0 Nov 28 05:40 blog.txt
thor@jumphost ~$ ssh steve@stapp02 ls -l /opt/security/
total 0
-rw-rw-r--+ 1 root root 0 Nov 28 05:40 story.txt
thor@jumphost ~$ ssh banner@stapp03 ls -l /opt/security/
total 0
-rw-rw-r--+ 1 root root 0 Nov 28 05:40 media.txt
thor@jumphost ~$ 
```

---

## 🎉 Outcome  

✔ File creation automated  
✔ ACLs applied correctly  
✔ Root ownership maintained  
✔ Permissions unique per server  
✔ Fully idempotent Ansible automation  
✔ Beginner-friendly and scalable solution  

---

## 🧠 Key Learnings

- **ACLs offer more flexibility than traditional permissions:** With ACLs, you can assign specific permissions to individual users or groups, not just the owner or group.
- **Ansible's ansible.posix.acl module makes ACL automation easy:** It ensures consistent ACL settings across all servers — no manual setfacl commands needed.
- **File ownership and ACLs complement each other:** Root still owns the files, guaranteeing security, while ACLs allow controlled access.
- **ABreaking tasks into separate host blocks increases clarity:** Each server’s unique ACL requirement is cleanly handled in its own section.
- **Always test connectivity before running playbooks:** A simple ansible -m ping saves time and helps detect SSH issues early.

---

### Official Ansible Documentation Links:
  - [File Module](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/file_module.html)
  - [ACL Module](https://docs.ansible.com/projects/ansible/latest/collections/ansible/posix/acl_module.html)

>ACL management is a crucial skill for Linux-based deployments, and automating it with Ansible takes you one step closer to mastering infra-as-code.
