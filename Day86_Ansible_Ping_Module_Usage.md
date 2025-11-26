# 🖧 Day 86: Using the Ansible Ping Module  

## 🎯 Objective

The Nautilus DevOps team wants to verify SSH connectivity and ensure Ansible communication with all application servers in the Stratos Datacenter.  
To validate Ansible setup, the `ping` module will be used.

This task involves:
- Creating an inventory file on the jump host  
- Verifying connectivity to all app servers  
- Ensuring Ansible is configured properly before running advanced playbooks  

---

## 📁 Task Breakdown

### 1. Create the Inventory File

Location:  
`~/playbook/inventory`

```ini
[app_servers]
stapp01 ansible_host=172.16.238.10 ansible_user=tony ansible_ssh_private_key_file=~/.ssh/id_rsa ansible_ssh_common_args='-o StrictHostKeyChecking=no'
stapp02 ansible_host=172.16.238.11 ansible_user=steve ansible_ssh_private_key_file=~/.ssh/id_rsa ansible_ssh_common_args='-o StrictHostKeyChecking=no'
stapp03 ansible_host=172.16.238.12 ansible_user=banner ansible_ssh_private_key_file=~/.ssh/id_rsa ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

### 🔑 Configure SSH Access

Generate SSH key and share it with all app servers:
```bash
ssh-keygen
ssh-copy-id tony@stapp01
ssh-copy-id steve@stapp02
ssh-copy-id banner@stapp03
```
### 🧪 Test Connectivity Using the Ansible Ping Module

Run the following command:  
```bash
ansible -i ~/playbook/inventory all -m ping
```

Expected Output:  

```javascript
stapp01 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}

stapp02 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}

stapp03 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```
### ✔ This confirms

- SSH authentication works
- Inventory is correctly configured
- Ansible can reach all the nodes

---

## 📘 Learnings From Day 86

### Key Takeaways

- The Ansible ping module is not ICMP ping — it checks Python connectivity on remote hosts.
- It's the simplest and most reliable way to confirm Ansible communication.
- If ping fails, future playbooks will fail too — this is the earliest warning signal.

### Best Practices

- Always run the ping test after creating or updating inventory.
- Use SSH keys, not passwords.
- Keep inventory grouped cleanly (e.g., app_servers).
- Enable StrictHostKeyChecking appropriately for automation nodes.

---

## 🔚 Closing Notes

Day 86 reinforces the foundation of automation - reliable connectivity.
Before deploying apps, copying files, or installing packages, always validate with a simple ping check.

>A strong foundation leads to strong automation 🚀🔥

---

## 👨‍💻 Author  
**Vamshi Krishna**  
[Connect on LinkedIn](https://in.linkedin.com/in/vamshi7)  
Passionate about DevOps, infrastructure automation, CI/CD, cloud, and SRE practices.  

---
