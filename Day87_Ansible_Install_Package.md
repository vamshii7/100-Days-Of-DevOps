# 📦 Day 87: Install Packages on App Servers Using Ansible
## 🎯 Objective
The Nautilus Application Development team needs to install prerequisites on all application servers in the Stratos Datacenter. Since the DevOps team already uses Ansible for automation, package installation must be handled with a centralized playbook.  

<div align="center">
<img width="691" height="588" alt="Screenshot 2025-11-26 125850" src="https://github.com/user-attachments/assets/11f4c8a7-9d37-4dad-9be6-0aadb420bbdf" />

</div>  

Today’s goal:
- Create an inventory  
- Write a playbook  
- Install `wget` on all app servers  
- Ensure user `thor` can run the playbook without extra arguments

---

## ⚙️Steps Performed

### 1. Generate SSH Keys on Jump Host and Configure Access  

``` bash
ssh-keygen
ssh-copy-id tony@stapp01
ssh-copy-id steve@stapp02
ssh-copy-id banner@stapp03
```
  Ensure users have sudo privileges

### 2. Create Inventory File

Path: `/home/thor/playbook/inventory`

```ini
[app_servers]
stapp01 ansible_host=172.16.238.10 ansible_user=tony ansible_ssh_private_key_file=~/.ssh/id_rsa ansible_ssh_common_args='-o StrictHostKeyChecking=no'
stapp02 ansible_host=172.16.238.11 ansible_user=steve ansible_ssh_private_key_file=~/.ssh/id_rsa ansible_ssh_common_args='-o StrictHostKeyChecking=no'
stapp03 ansible_host=172.16.238.12 ansible_user=banner ansible_ssh_private_key_file=~/.ssh/id_rsa ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

### 3. Create Ansible Playbook

Location: `/home/thor/playbook/playbook.yml`

``` yaml
---
- hosts: app_servers
  become: yes
  tasks:
    - name: Install wget package
      yum:
        name: wget
        state: installed
```

### 🔑 Explanation
- `hosts: app_servers` → targets the group [appservers] defined in your inventory file.
- `become: yes` → runs the task with sudo privileges (needed for package installation).
- `yum module` → ensures the package is installed via YUM (works on RHEL/CentOS).
- `state: present/installed` → installs the package if missing, but doesn’t reinstall if already present.  

---

## 📡Connectivity Test

### Test SSH Connectivity with Ansible

``` bash
ansible -i /home/thor/playbook/inventory all -m ping
```

Expected output: All nodes should return "pong".  

<img width="678" height="423" alt="Screenshot 2025-11-26 131155" src="https://github.com/user-attachments/assets/826fab98-548b-4478-9fe4-4bf69a337e5d" />
<br>  

---

## ▶️ Execute the Playbook

```bash
ansible-playbook -i /home/thor/playbook/inventory /home/thor/playbook/playbook.yml
```  
<img width="869" height="365" alt="Screenshot 2025-11-26 131612" src="https://github.com/user-attachments/assets/5ea2cab1-6fd8-4bdc-ab64-ffc7954d366d" />
<br>  

---

## ✔️ Verify Installation

```bash
ssh tony@stapp01 wget --version | head -n 1
ssh steve@stapp02 wget --version | head -n 1
ssh banner@stapp03 wget --version | head -n 1
```

<img width="542" height="177" alt="Screenshot 2025-11-26 132312" src="https://github.com/user-attachments/assets/36ab1b5a-d37b-49ad-baf4-11e32f3c2250" />
<br>  

---

## 🧠 Learnings

### Key Takeaways  
- Ansible inventory structuring is foundational for automation.
- yum module simplifies OS-level package handling.
- SSH key distribution enables passwordless orchestration.
- become: yes is required for privileged tasks.
- Proper sudo privileges are crucial for package installation.  

### Best Practices
- Keep inventory files grouped and readable.
- Validate connectivity before running playbooks.
- Use minimal privilege escalation.
- Store SSH keys securely.
- Always verify installation manually.  

---

## 🔚 Closing Notes

Automating package installations ensures consistency, speeds up workflows, and builds reliability across distributed environments. Each automated task strengthens your DevOps fundamentals.  

---

## 👨‍💻 Author  
**Vamshi Krishna**  
[Connect on LinkedIn](https://in.linkedin.com/in/vamshi7)  
Passionate about DevOps, infrastructure automation, CI/CD, cloud, and SRE practices.  

---
