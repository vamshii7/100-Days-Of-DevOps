# 🚀Day 82: Create Ansible Inventory for App Server Testing

> Ever wondered how a single inventory file can make or break your entire Ansible automation flow? Today was one of those days where a tiny file played a massive role.

## 🌐Overview
The Nautilus DevOps team is testing various Ansible playbooks across their application stack. For today's task, the goal was to configure an Ansible inventory so that the jump host can successfully run a playbook on App Server 3 in the Stratos DC environment.  

<img width="612" height="586" alt="Screenshot 2025-11-20 140255" src="https://github.com/user-attachments/assets/b2b6ad2c-b4cd-4224-9718-f76eca8dd3f1" />  

<br>  
 
This challenge focuses on creating the required INI based inventory, enabling passwordless SSH access, and validating the playbook execution that installs and starts the httpd service.

---

## 📋Task Requirements

### Create an Ansible inventory file
Location:
```
/home/thor/playbook/inventory
```

### Include App Server 3
The hostname must match the Stratos DC naming convention. Example:
- stapp01 for App Server 1
- stapp02 for App Server 2
- stapp03 for App Server 3

### Provide correct variables
- Host IP
- SSH user
- SSH private key path

---

## 🛠️Steps Performed

### 1. Generate SSH key on the jump host
```
ssh-keygen -t rsa
```

### 2. Copy key to App Server 3
```
ssh-copy-id banner@172.16.238.12
```

### 3. Create the Ansible Inventory File  
Path:
```
/home/thor/playbook/inventory
```

Content:  
```ini
[app_servers]
stapp03 ansible_host=172.16.238.12 ansible_user=banner ansible_ssh_private_key_file=~/.ssh/id_rsa
```

### 4. Run the playbook  
```
ansible-playbook -i inventory playbook.yml
```  

<img width="943" height="285" alt="Screenshot 2025-11-20 140543" src="https://github.com/user-attachments/assets/045b0eaa-891d-4e14-adc2-faf30dc5a43b" />  

<br>  

The playbook executed successfully, installing and starting the httpd service on App Server 3.

### 5. Verify on App Server  
```
sudo systemctl status httpd
```

<img width="966" height="488" alt="Screenshot 2025-11-20 140826" src="https://github.com/user-attachments/assets/157a4682-a6d1-4405-9834-49461913c54a" />  

<br>  
The service was confirmed running.

---

## Playbook Used

```yaml
---
- hosts: all
  become: yes
  become_user: root
  tasks:
    - name: Install httpd package
      yum:
        name: httpd
        state: installed

    - name: Start service httpd
      service:
        name: httpd
        state: started
```

---

## 💡Key Learnings

### 1. Ansible Inventory is the foundation of automation
A well structured inventory removes ambiguity and ensures seamless node targeting.

### 2. Passwordless SSH improves reliability
Automation should not rely on interactive password prompts. SSH keys are essential.

### 3. Hostnames matter
Using stapp03 instead of only the IP improves clarity and aligns with naming conventions.

### 4. Minimal arguments equal better CI/CD
Design your inventory so that commands such as the following work without extra flags:
```
ansible-playbook -i inventory playbook.yml
```

---

## 🧰Best Practices
- Keep inventory files simple and modular.
- Use groups like [web], [db], and [app_servers] for scalability.
- Always test SSH connectivity using:  
  ```
  ansible all -m ping -i inventory
  ```
- Store sensitive keys securely. Avoid committing them to Git.
- Follow consistent naming conventions.
- Validate with check mode before running destructive changes.

---

## 🎯Final Thoughts  
This task highlights how crucial the inventory file is in Ansible automation. Even a perfect playbook fails if the inventory is misconfigured. By setting up SSH keys, defining hosts properly, and validating playbook execution, you build the foundation needed for more advanced orchestrations.  

This exercise helps beginners understand how Ansible communicates with remote nodes, which is a key skill for any DevOps engineer.  
---  


