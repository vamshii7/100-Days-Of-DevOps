# 🚀 Day 88: Ansible Blockinfile Module  
Automating Web Server Setup & Content Deployment  

<img width="811" height="807" alt="Screenshot 2025-11-27 165119" src="https://github.com/user-attachments/assets/28698e69-8b58-4360-abfa-32f5b39e422f" />


---

## 📌 **Task Overview**

The Nautilus DevOps team needs to install and configure a basic `httpd` web server across all application servers in the Stratos Datacenter.  
Along with installation, a sample webpage must be deployed using **Ansible only**, ensuring there is no manual intervention.

You are required to:

1. Install **httpd** on all app servers.
2. Ensure the **httpd service** is enabled and running.
3. Use **blockinfile** module to add predefined content to `/var/www/html/index.html`.
4. Set file **owner/group** to `apache`.
5. Set file **permissions** to `0744`.
6. Ensure playbook runs directly using: `ansible-playbook -i inventory playbook.yml`

---

## 🛠 **Solution Steps**

### 1️⃣ Generate SSH Keys & Share with App Servers

On the Jump Host (Control Node):  

```bash
ssh-keygen
ssh-copy-id tony@stapp01
ssh-copy-id steve@stapp02
ssh-copy-id banner@stapp03
```

### 2️⃣ Configure Passwordless Privilege Escalation

Login to each app server and update sudoers to allow installing packages, managing services, and file ownership changes.


### 📁 Inventory Configuration

Create or update the inventory file: `/home/thor/ansible/inventory`  

```ini
[app_servers]
stapp01 ansible_host=172.16.238.10 ansible_user=tony ansible_ssh_private_key_file=~/.ssh/id_rsa ansible_ssh_common_args='-o StrictHostKeyChecking=no'
stapp02 ansible_host=172.16.238.11 ansible_user=steve ansible_ssh_private_key_file=~/.ssh/id_rsa ansible_ssh_common_args='-o StrictHostKeyChecking=no'
stapp03 ansible_host=172.16.238.12 ansible_user=banner ansible_ssh_private_key_file=~/.ssh/id_rsa ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```  

### 📜 Ansible Playbook

Create the playbook at: `/home/thor/ansible/playbook.yml`  

```yml
---
- hosts: app_servers
  become: yes

  tasks:
    - name: Install httpd web server
      ansible.builtin.package:
        name: httpd
        state: present

    - name: Ensure httpd service is enabled and running
      ansible.builtin.service:
        name: httpd
        state: started
        enabled: yes

    - name: Add content to /var/www/html/index.html
      ansible.builtin.blockinfile:
        path: /var/www/html/index.html
        block: |
          Welcome to XfusionCorp!
          This is Nautilus sample file, created using Ansible!
          Please do not modify this file manually!
        create: yes
        owner: apache
        group: apache
        mode: '0744'
```
---

## 📡 Connectivity Check  

```bash
ansible -i /home/thor/ansible/inventory all -m ping
```

Expected output: 

<img width="697" height="426" alt="Screenshot 2025-11-27 170403" src="https://github.com/user-attachments/assets/729b7e67-0274-4d03-84cd-f428313c37ac" />
<br>

---

## ▶️ Run the Playbook
```bash
ansible-playbook -i /home/thor/ansible/inventory /home/thor/ansible/playbook.yml
```

<img width="982" height="565" alt="Screenshot 2025-11-27 171333" src="https://github.com/user-attachments/assets/823ee6d1-d31a-4f07-8246-37e461470826" />
<br>

---

## 🔍 Verification
### 1. Check file permissions & ownership
```bash
ssh tony@stapp01 ls -l /var/www/html/
ssh steve@stapp02 ls -l /var/www/html/
ssh banner@stapp03 ls -l /var/www/html/
```
### 2. Check file content
```bash
ssh tony@stapp01 cat /var/www/html/index.html
ssh steve@stapp02 cat /var/www/html/index.html
ssh banner@stapp03 cat /var/www/html/index.html
```

<img width="765" height="511" alt="Screenshot 2025-11-27 172148" src="https://github.com/user-attachments/assets/f6dca7e5-fe75-4e25-b402-054b33d26152" />

<br>

---

## 🎉 Outcome

✔ Httpd installed on all app servers  
✔ Service enabled and running  
✔ Web page deployed using Ansible-only automation  
✔ Permissions, ownership, and content set correctly  
✔ Playbook validated to run exactly as required  

---

## 🧠 Key Learnings
- **blockinfile is perfect for configuration & content management:** It ensures structured text is inserted cleanly without overwriting or duplicating content.
- **Idempotency is the backbone of Ansible:** Running the same playbook multiple times always results in a predictable state.
- **Using inventory variables improves clarity:** Hostname, SSH key path, and connection settings stay neatly organized.
- **Always verify service state after installs:** Installing packages isn't enough - ensuring services are running is crucial.
- **Automation improves consistency across environments:** Instead of updating multiple servers manually, one playbook does it all reliably.

---

## 🧩 Best Practices

✔ Use become only when required to reduce privilege misuse  
✔ Always keep SSH keys secure and restricted  
✔ Validate inventory connectivity using ansible -m ping before running playbooks  
✔ Keep playbooks modular - separate tasks logically  
✔ Test on one host first when writing new modules  
✔ Use explicit file permissions for predictable security  
✔ Prefer blockinfile or templates instead of raw shell commands  
✔ Document every step - it helps when revisiting configs later  

---

>Another successful automation task completed in the #100DaysOfDevOps journey! 🚀

---

## 👨‍💻 Author  
**Vamshi Krishna**  
DevOps Engineer | DevOps & Kubernetes Enthusiast  
[Connect on LinkedIn](https://in.linkedin.com/in/vamshi7)  
> ⚙️ _Feel free to fork and contribute — PRs are welcome!_
---
