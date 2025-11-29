# Day 92: Managing Jinja2 Templates Using Ansible

## 📝 Introduction

Jinja2 templates allow dynamic file generation in Ansible. Instead of
hardcoding server‑specific values, templates enable flexible content
rendering using variables such as `inventory_hostname`. This ensures
automation remains scalable, reusable, and environment‑independent.

------------------------------------------------------------------------

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

------------------------------------------------------------------------

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

-   https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template_module.html\
-   https://jinja.palletsprojects.com

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