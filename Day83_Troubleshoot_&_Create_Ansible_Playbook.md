# Day 83: Troubleshoot and Create Ansible Playbook  

<img width="803" height="490" alt="Screenshot 2025-11-20 155457" src="https://github.com/user-attachments/assets/00e4e7e8-b780-4c98-9d85-151d10a832f6" />


------------------------------------------------------------------------

## ✅ Task Summary

1.  Fix the inventory file located at:

        /home/thor/ansible/inventory

    It must target **App Server 2** in Stratos DC.

2.  Create a playbook:

        /home/thor/ansible/playbook.yml

    The playbook must create an empty file: `/tmp/file.txt` on App Server 2.

------------------------------------------------------------------------

## 🔧 Solution Steps

### **1. Generate SSH Key**

Generate an SSH key on the jump host:

``` bash
ssh-keygen -t rsa
```

### **2. Copy SSH Key to App Server 2**

``` bash
ssh-copy-id steve@172.16.238.11
```

------------------------------------------------------------------------

## 📝 Update the Inventory File

Edit:

    /home/thor/ansible/inventory

Add the correct entry for App Server 2:

    stapp02 ansible_host=172.16.238.11 ansible_user=steve ansible_ssh_common_args='-o StrictHostKeyChecking=no' ansible_ssh_private_key_file=~/.ssh/id_rsa

Save the file.

------------------------------------------------------------------------

## 📘 Create the Playbook

Create:

    /home/thor/ansible/playbook.yml

Add:

``` yaml
- hosts: stapp02
  become: yes
  tasks:
    - name: Create an empty file
      file:
        path: /tmp/file.txt
        state: touch
```

### **🔍 Explanation**

-   **file:** Uses Ansible's file module\
-   **path:** Path of the file on the target server\
-   **state: touch:** Creates the file if missing, updates timestamp if
    it already exists\
-   **become: yes:** Run with elevated privileges

------------------------------------------------------------------------

## 🧪 Validate Connectivity

Test SSH/Ansible communication:

``` bash
ansible -i inventory all -m ping
```

------------------------------------------------------------------------

## ▶️ Run the Playbook

``` bash
ansible-playbook -i inventory playbook.yml
```

<img width="917" height="444" alt="Screenshot 2025-11-20 155723" src="https://github.com/user-attachments/assets/80a6e8e6-7390-42a3-abee-e226dc3b29ce" />  

------------------------------------------------------------------------

## 🔎 Verify on App Server 2

SSH into the server:

``` bash
ssh steve@stapp02
```

Check if the file was created:

``` bash
ls -l /tmp/file.txt
```

<img width="894" height="178" alt="Screenshot 2025-11-20 155924" src="https://github.com/user-attachments/assets/f5aefa40-7cbc-4c24-bc4a-9410c36bad62" />  

------------------------------------------------------------------------

🎉 **Playbook executed successfully and file created!**
