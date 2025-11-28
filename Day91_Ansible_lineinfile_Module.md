# 🚀 Day 91: Ansible `lineinfile` Module

Automating Web Server Deployment & Content Management

------------------------------------------------------------------------
<div align="center">
<img width="726" height="777" alt="Screenshot 2025-11-28 151631" src="https://github.com/user-attachments/assets/1c71250e-283f-4b19-bac4-2ef06941d607" />
</div>
<br>

------------------------------------------------------------------------

## 📌 **Task Overview**

The Nautilus DevOps team needs a simple **httpd web server** deployed and configured across all application servers in the Stratos Datacenter.  

Additionally, they want a **sample webpage** deployed and updated automatically using the **Ansible `lineinfile` module**.  

Your mission:

✔ Install and start **httpd** on all app servers  
✔ Create `/var/www/html/index.html` with default content  
✔ Add a new line at the **top** of the file using `lineinfile`  
✔ Ensure file ownership and permissions match requirements  
✔ Automate everything through a single Ansible playbook  

------------------------------------------------------------------------

## 🛠 **Prerequisites**

### 1️⃣ Generate SSH keys on Jump Host

``` bash
ssh-keygen
ssh-copy-id tony@stapp01
ssh-copy-id steve@stapp02
ssh-copy-id banner@stapp03
```

SSH into each app server and add the respective users to the sudoers file to allow package installations and permission modifications.  

------------------------------------------------------------------------

## 📁 **Inventory File**

Create/update inventory at: `/home/thor/ansible/inventory`  

``` ini
[app_servers]
stapp01 ansible_host=172.16.238.10 ansible_user=tony ansible_ssh_private_key_file=~/.ssh/id_rsa ansible_ssh_common_args='-o StrictHostKeyChecking=no'
stapp02 ansible_host=172.16.238.11 ansible_user=steve ansible_ssh_private_key_file=~/.ssh/id_rsa ansible_ssh_common_args='-o StrictHostKeyChecking=no'
stapp03 ansible_host=172.16.238.12 ansible_user=banner ansible_ssh_private_key_file=~/.ssh/id_rsa ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

------------------------------------------------------------------------

## 📜 **Ansible Playbook -- `/home/thor/ansible/playbook.yml`**

``` yml
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

    - name: Create /var/www/html/index.html with initial content
      ansible.builtin.copy:
        dest: /var/www/html/index.html
        content: "This is a Nautilus sample file, created using Ansible!\n"
        owner: apache
        group: apache
        mode: '0744'

    - name: Add line at the top of /var/www/html/index.html
      ansible.builtin.lineinfile:
        path: /var/www/html/index.html
        line: "Welcome to xFusionCorp Industries!"
        insertbefore: BOF
        owner: apache
        group: apache
        mode: '0744'
```

------------------------------------------------------------------------

## 📡 **Connectivity Check**

``` bash
ansible -i /home/thor/ansible/inventory all -m ping
```  

<img width="535" height="398" alt="Screenshot 2025-11-28 151647" src="https://github.com/user-attachments/assets/7dd3703e-c936-4143-a723-89e08be1f7aa" />


------------------------------------------------------------------------

## ▶️ **Run the Playbook**

``` bash
ansible-playbook -i /home/thor/ansible/inventory /home/thor/ansible/playbook.yml
```  

<img width="929" height="641" alt="Screenshot 2025-11-28 152011" src="https://github.com/user-attachments/assets/9b21d93c-8e6b-4814-831d-91baed5cdbef" />


------------------------------------------------------------------------

## 🔍 **Verification**

### ✔ Check httpd status

``` bash
ssh tony@stapp01 sudo systemctl status httpd | head -10
```  

### ✔ Check file content

``` bash
ssh tony@stapp01 cat /var/www/html/index.html
```  

### ✔ Check file permissions

``` bash
ssh tony@stapp01 ls -l /var/www/html/index.html
```

Expected output:

<img width="871" height="794" alt="Screenshot 2025-11-28 152441" src="https://github.com/user-attachments/assets/0f8f56c2-6c26-49f4-8e6f-d8eb7fb0158e" />

------------------------------------------------------------------------

## 🧠 **Key Learnings**

### 1. `lineinfile` lets you surgically edit files  
  - Perfect for updating configs or injecting content without overwriting the file.

### 2. `insertbefore: BOF` is a powerful option  
  - Useful when adding headers, banners, alerts, or compliance text at the top of files.

### 3. Using `copy` first + `lineinfile` second is a reliable pattern  
  - Ensures deterministic content with idempotency.

### 4. Ownership & permissions matter  
  - Web servers like Apache often require correct file ownership (`apache:apache`) and modes (`0744`) to avoid access issues.

### 5. Consistency across servers  
  - Ansible ensures all nodes receive the **exact same configuration**, eliminating drift.

------------------------------------------------------------------------

## ⭐ Best Practices Followed

- Separate inventory for cleaner host management
- `package` module used instead of `yum` for OS portability
- Idempotent tasks --- running multiple times produces the same result
- SSH key authentication vs password → more secure & faster
- Service enabled at boot (`enabled: yes`)
- No unmanaged manual changes on nodes  

------------------------------------------------------------------------

## 🎉 **Outcome**

✔ httpd installed & running  
✔ Sample webpage deployed  
✔ Additional content added using `lineinfile` at BOF  
✔ Ownership and permissions correctly applied  
✔ Fully automated, repeatable, production-ready playbook  

------------------------------------------------------------------------

## 🔗 **Official Ansible Documentation**

-   [`lineinfile`Module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/lineinfile_module.html)
-   [`copy`Module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html)
-   [`service`Module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/service_module.html)

------------------------------------------------------------------------

> Day 91 reinforces that automation isn't just about deploying services **it's about enforcing consistency, eliminating drift, and managing configuration intelligently.**
