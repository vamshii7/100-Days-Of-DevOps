# Day 92: Managing Jinja2 Templates Using Ansible

## 📝 Introduction

Jinja2 templates allow dynamic file generation in Ansible. Instead of
hardcoding server‑specific values, templates enable flexible content
rendering using variables such as `inventory_hostname`. This ensures
automation remains scalable, reusable, and environment‑independent.

------------------------------------------------------------------------

## 📌 Task

One of the Nautilus DevOps team members is working on to develop a role for httpd installation and configuration. Work is almost completed, however there is a requirement to add a jinja2 template for index.html file. Additionally, the relevant task needs to be added inside the role. The inventory file ~/ansible/inventory is already present on jump host that can be used. Complete the task as per details mentioned below:

a. Update `~/ansible/playbook.yml` playbook to run the httpd role on `App Server 1`.

b. Create a jinja2 template `index.html.j2` under `/home/thor/ansible/role/httpd/templates/` directory and add a line This file was created using Ansible on <respective server> (for example This file was created using Ansible on stapp01 in case of App Server 1). Also please make sure not to hard code the server name inside the template. Instead, use `inventory_hostname` variable to fetch the correct value.

c. Add a task inside `/home/thor/ansible/role/httpd/tasks/main.yml` to copy this template on App Server 1 under `/var/www/html/index.html`. Also make sure that `/var/www/html/index.html` file's permissions are `0644`.

d. The `user/group` owner of `/var/www/html/index.html` file must be respective sudo user of the server (for example tony in case of stapp01).  

---
## 📌 Task Requirements  

### a. Update `~/ansible/playbook.yml`

Run the `httpd` role **only on App Server 1**:

``` yaml
---
- hosts: stapp01
  become: yes
  roles:
    - httpd
```

------------------------------------------------------------------------

### b. Create Jinja2 Template

File: `/home/thor/ansible/role/httpd/templates/index.html.j2`

``` jinja2
This file was created using Ansible on {{ inventory_hostname }}
```

This uses `inventory_hostname` dynamically --- **no hardcoding**.

------------------------------------------------------------------------

### c. Add Task Inside Role

File: `/home/thor/ansible/role/httpd/tasks/main.yml`

``` yaml
---
# tasks file for role/httpd

- name: install the latest version of HTTPD
  yum:
    name: httpd
    state: latest

- name: Start service httpd
  service:
    name: httpd
    state: started

- name: Copy Template on App Server
  template:
    src: index.html.j2
    dest: /var/www/html/index.html
    mode: '0644'
    owner: "{{ ansible_user }}"
    group: "{{ ansible_user }}"
```

This ensures: - Correct permissions (`0644`) - Correct owner/group using
dynamic Ansible facts (`ansible_user`)

------------------------------------------------------------------------

## 🔍 Connectivity Test

``` bash
ansible -i inventory all -m ping
```

------------------------------------------------------------------------

## ▶️ Run the Playbook

``` bash
ansible-playbook -i inventory playbook.yml
```

<img width="860" height="382" alt="Screenshot 2025-11-29 120037" src="https://github.com/user-attachments/assets/f8cda732-ca94-428a-b04d-b227d7d1b4a7" />
<br>

------------------------------------------------------------------------

## 🔍 Verification
### ✔ Check by running below command
```bash
curl stapp01
```

<img width="629" height="66" alt="Screenshot 2025-11-29 120129" src="https://github.com/user-attachments/assets/0edce5d8-527e-4175-9000-c274f07194da" />
<br>

---

## 🎯 Key Learnings

-   Jinja2 templates help avoid hardcoding.
-   `inventory_hostname` ensures correct server‑specific rendering.
-   Roles keep automation modular and reusable.
-   Dynamic ownership using `ansible_user` prevents issues across multiple servers.  

------------------------------------------------------------------------

## 🛠 Best Practices

-   Always separate templates inside `templates/` directory.
-   Keep roles modular: **tasks**, **handlers**, **templates**,**vars**, **defaults**.
-   Use meaningful, readable variable names.
-   Validate templates using `ansible --syntax-check`.
-   Test roles in isolation before integration.

------------------------------------------------------------------------

## 🏁 Outcome

✔ HTTPD successfully installed  
✔ Template deployed dynamically  
✔ Permissions and ownership set correctly  
✔ Playbook works without extra arguments  

------------------------------------------------------------------------

## 📚 Official Documentation

-   [Ansible Tamplating(Jinja2)](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_templating.html#jinja2-example)

------------------------------------------------------------------------

## 💭 Final Thoughts

Mastering templates is a crucial step in becoming an automation‑first DevOps engineer.  
Dynamic files = scalable infrastructure.  

---

>Another successful automation task completed in the #100DaysOfDevOps journey! 🚀

---

## 👨‍💻 Author  
**Vamshi Krishna**  
DevOps Engineer | DevOps & Kubernetes Enthusiast  
[Connect on LinkedIn](https://in.linkedin.com/in/vamshi7)  
> ⚙️ _Feel free to fork and contribute — PRs are welcome!_
---
