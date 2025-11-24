# 🚀 Day 84: Copy Data to Application Servers using Ansible

## 📘Overview  

<img width="704" height="497" alt="Screenshot 2025-11-24 171934" src="https://github.com/user-attachments/assets/0ed071d0-9789-45d3-9505-7653a4cb5bb6" />
<br>  

The Nautilus DevOps team needed to copy data from the jump host to all
application servers in the Stratos DC environment using Ansible.\
This task involved setting up SSH access, configuring an inventory,
writing a playbook, and validating the file transfer.

------------------------------------------------------------------------

## 🗂️Task Requirements

### a. Create Ansible Inventory

Path:

    /home/thor/ansible/inventory

Include all application servers as managed nodes.

### b. Create Ansible Playbook

Path:

    /home/thor/ansible/playbook.yml

The playbook must copy the file:

    /usr/src/dba/index.html

to all application servers at:

    /opt/dba

Validation will run:

    ansible-playbook -i inventory playbook.yml

No extra arguments should be required.

------------------------------------------------------------------------

## ⚙️Steps Performed

### 1. Generate SSH Keys on Jump Host and Configure Access

``` bash
ssh-keygen
ssh-copy-id tony@stapp01
ssh-copy-id steve@stapp02
ssh-copy-id banner@stapp03
```

### 2. Create Inventory File

Location: `/home/thor/ansible/inventory`

``` ini
[app_servers]
stapp01 ansible_host=172.16.238.10 ansible_user=tony ansible_ssh_private_key_file=~/.ssh/id_rsa
stapp02 ansible_host=172.16.238.11 ansible_user=steve ansible_ssh_private_key_file=~/.ssh/id_rsa
stapp03 ansible_host=172.16.238.12 ansible_user=banner ansible_ssh_private_key_file=~/.ssh/id_rsa
```

### 3. Create Ansible Playbook

Location: `/home/thor/ansible/playbook.yml`

``` yaml
---
- name: Copy file to all app servers
  hosts: app_servers
  become: true
  tasks:
    - name: Copy index.html to /opt/dba
      ansible.builtin.copy:
        src: /usr/src/dba/index.html
        dest: /opt/dba
```

------------------------------------------------------------------------

## 📡Connectivity Test

### Test SSH Connectivity with Ansible

``` bash
ansible -i inventory all -m ping
```

Expected output: All nodes should return "pong".  

<img width="621" height="456" alt="Screenshot 2025-11-24 172943" src="https://github.com/user-attachments/assets/fcec32b3-f917-4871-9d92-c411b92a3094" />
<br>

------------------------------------------------------------------------

## Execute the Playbook

``` bash
ansible-playbook -i inventory playbook.yml
```

<img width="875" height="371" alt="Screenshot 2025-11-24 173207" src="https://github.com/user-attachments/assets/a8512331-2782-450c-913e-c23d70ecfadc" />
<br>

The playbook runs without additional arguments as required.

------------------------------------------------------------------------

## 🔍Verification

Log into each application server and verify the copied file:

``` bash
ls -l /opt/dba
```

Expected:\
`index.html` must be present.  

<img width="766" height="266" alt="Screenshot 2025-11-24 173537" src="https://github.com/user-attachments/assets/bace21df-5668-4fb0-a502-0ee0ed6c3040" />
<br>

------------------------------------------------------------------------

## 🧠Key Learnings

### 1. Inventory grouping improves scalability

Using a group such as `[app_servers]` makes multi-host automation
efficient.

### 2. Copy module is simple and powerful

`ansible.builtin.copy` is ideal for transferring local files to remote
machines.

### 3. Minimalistic playbooks help in CI/CD

A properly structured inventory ensures commands work without extra
flags.

### 4. Passwordless SSH is essential

Automations should never rely on manual password entry.

------------------------------------------------------------------------

## 🏁Final Thoughts

This task reinforces the importance of clean inventory management and
simple yet powerful Ansible playbooks.\
By ensuring SSH access, preparing a structured inventory, and executing
the file transfer playbook, we achieve consistent and reliable
automation across multiple servers.

------------------------------------------------------------------------
