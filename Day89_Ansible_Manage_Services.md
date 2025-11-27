# 🚀 Day 89: Ansible Manage Services  
Automating Web Server installation and startup.

<img width="738" height="610" alt="Screenshot 2025-11-27 204013" src="https://github.com/user-attachments/assets/6066408f-9128-4ff3-a940-c02cbaf439ad" />

---

## 📌 **Task Overview**

The Nautilus DevOps team needs to install and configure a basic `httpd` web server across all application servers in the Stratos Datacenter.  
Along with installation, make sure to start and enable httpd service on all app servers, ensuring there is no manual intervention.

You are required to:

1. Install **httpd** on all app servers.
2. Ensure the **httpd service** is enabled and running.
3. Ensure playbook runs directly using: `ansible-playbook -i inventory playbook.yml`

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
### Check the httpd status on all app servers
```bash
ssh tony@stapp01 ls -l /var/www/html/
ssh steve@stapp02 ls -l /var/www/html/
ssh banner@stapp03 ls -l /var/www/html/
```

<img width="984" height="623" alt="Screenshot 2025-11-27 210236" src="https://github.com/user-attachments/assets/8d8b58ec-b3e1-4679-8c05-25ce3684997e" />

<br>

---

## 🎉 Outcome

✔ Httpd installed on all app servers  
✔ Service enabled and running   
✔ Playbook validated to run exactly as required  

---

## 🧠 Key Learnings

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
