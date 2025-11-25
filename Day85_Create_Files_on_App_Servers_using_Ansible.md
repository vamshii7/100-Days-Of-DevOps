# Day 85 - Create Files on App Servers Using Ansible  
![Ansible](https://img.shields.io/badge/Automation-Ansible-blue) ![DevOps](https://img.shields.io/badge/DevOps-100DaysOfDevOps-success) ![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

---

## 🚀 **Automating File Management Across Servers with Ansible**

Day 85 of my 100 Days of DevOps journey dives into one of the simplest yet most powerful Ansible capabilities:  
**remote file creation with precise ownership and permissions** across multiple servers.

This task demonstrates how automation eliminates repetitive server-by-server actions and replaces them with a clean, efficient, and scalable playbook.  

<img width="718" height="598" alt="Screenshot 2025-11-25 143904" src="https://github.com/user-attachments/assets/cd1bc80b-d519-4955-a771-3098e8feec38" />
<br>

---

# 📌 **Task Summary**

The objective for today:

### ✔ Create an inventory file  
### ✔ Create a playbook to generate a file on all app servers  
### ✔ Set ownership and permissions uniquely per server  
### ✔ Ensure the playbook runs exactly with:  
```
ansible-playbook -i inventory playbook.yml
```

---

# 🗂 **Step 1: Generate SSH Keys & Copy to All App Servers**

```bash
ssh-keygen
ssh-copy-id tony@stapp01
ssh-copy-id steve@stapp02
ssh-copy-id banner@stapp03
```

---

# 🗂 **Step 2: Create the Inventory File**

📄 **File:** `~/playbook/inventory`

```ini
[app_servers]
stapp01 ansible_host=172.16.238.10 ansible_user=tony ansible_ssh_private_key_file=~/.ssh/id_rsa
stapp02 ansible_host=172.16.238.11 ansible_user=steve ansible_ssh_private_key_file=~/.ssh/id_rsa
stapp03 ansible_host=172.16.238.12 ansible_user=banner ansible_ssh_private_key_file=~/.ssh/id_rsa
```

---

# 📝 **Step 3: Create the Playbook**

📄 **File:** `~/playbook/playbook.yml`

```yaml
---
- hosts: app_servers
  become: yes
  tasks:
    - name: Create /home/app.txt on stapp01
      file:
        path: /home/app.txt
        state: touch
        owner: tony
        group: tony
        mode: '0744'
      when: inventory_hostname == "stapp01"

    - name: Create /home/app.txt on stapp02
      file:
        path: /home/app.txt
        state: touch
        owner: steve
        group: steve
        mode: '0744'
      when: inventory_hostname == "stapp02"

    - name: Create /home/app.txt on stapp03
      file:
        path: /home/app.txt
        state: touch
        owner: banner
        group: banner
        mode: '0744'
      when: inventory_hostname == "stapp03"
```

---

# 🔍 **Step 4: Verify Connectivity**

```bash
ansible -i ~/playbook/inventory all -m ping
```  

<img width="674" height="418" alt="Screenshot 2025-11-25 144459" src="https://github.com/user-attachments/assets/86a3f347-88d3-4ba5-af55-366c4ed5cbca" />
<br>

---

# ▶️ **Step 5: Run the Playbook**

```bash
ansible-playbook -i inventory playbook.yml
```

<img width="918" height="551" alt="Screenshot 2025-11-25 144510" src="https://github.com/user-attachments/assets/f6499cf3-5e6f-438c-a2ef-50fb378324aa" />
<br>

---

# 🔎 **Step 6: Verify File on Each Server**

```bash
ssh tony@stapp01 ls -l /home/app.txt
ssh steve@stapp02 ls -l /home/app.txt
ssh banner@stapp03 ls -l /home/app.txt
```

<img width="616" height="134" alt="Screenshot 2025-11-25 144851" src="https://github.com/user-attachments/assets/556f1986-073a-4671-91ac-e92cd4ccf698" />
<br>

---

# 🧠 **Key Learnings**

### 🔹 1. Conditional tasks offer fine-grained control  
Using `when:` ensures each host receives the correct owner and group.

### 🔹 2. Ansible’s file module is incredibly versatile  
Touch files, manage permissions, enforce state - all declaratively.

### 🔹 3. Clean inventory = clean automation  
A properly structured inventory eliminates confusion and scaling issues.

### 🔹 4. SSH key-based access is essential  
It ensures automation runs smoothly without interactive authentication blocks.

---

# 💡 **Best Practices**

✨ Use descriptive task names  
✨ Keep playbooks idempotent  
✨ Separate inventory for clarity  
✨ Validate server connectivity before execution  
✨ Apply principle of least privilege when using become

---

# 🎯 **Closing Thoughts**

Day 85 reinforces a powerful truth about DevOps:

> **Small tasks, when automated well, create massive operational efficiency.**

Simple file creation can quickly become complex at scale - but Ansible ensures consistency, reliability and zero manual overhead.

---

# 👨‍💻 Author  
**Vamshi Krishna**  
[Connect on LinkedIn](https://in.linkedin.com/in/vamshi7)  
Passionate about DevOps, infrastructure automation, CI/CD, cloud, and SRE practices.  

---

