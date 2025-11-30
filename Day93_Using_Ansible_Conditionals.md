# Day 93: Using Ansible Conditionals

In Day 93 of the **#100DaysOfDevOps** challenge, I explored one of Ansible’s key features - **conditionals**.  
Conditionals make automation smarter by allowing tasks to run only when specific criteria are met.  
This prevents unnecessary operations, reduces errors, and increases control in multi-server deployments.

---

## 📝 **Task**

<img width="699" height="757" alt="Screenshot 2025-11-29 162510" src="https://github.com/user-attachments/assets/aea011ac-8bdd-490f-8272-4656a01c7318" />

<br>

The Nautilus DevOps team wants to utilize **Ansible conditionals** to perform different copy actions depending on the target server.

An inventory file already exists at:

```
/home/thor/ansible/inventory
```

Create a playbook (`/home/thor/ansible/playbook.yml`) that:

### **1️⃣ App Server 1 (stapp01)**  
Copy `blog.txt` → `/opt/data/blog.txt`  
Owner: `tony`  
Group: `tony`  
Permissions: `0655`

### **2️⃣ App Server 2 (stapp02)**  
Copy `story.txt` → `/opt/data/story.txt`  
Owner: `steve`  
Group: `steve`  
Permissions: `0655`

### **3️⃣ App Server 3 (stapp03)**  
Copy `media.txt` → `/opt/data/media.txt`  
Owner: `banner`  
Group: `banner`  
Permissions: `0655`

We must use:
- `when:` conditional  
- `ansible_nodename` fact  
- Run play for **all hosts** → `hosts: all`  
- Playbook should work with:

```
ansible-playbook -i inventory playbook.yml
```

---

## ✅ **Solution**

### **Create the playbook**

`vi /home/thor/ansible/playbook.yml`

```yaml
---
- hosts: all
  become: yes
  tasks:

    - name: Copy blog.txt to App Server 1
      ansible.builtin.copy:
        src: /usr/src/data/blog.txt
        dest: /opt/data/blog.txt
        owner: tony
        group: tony
        mode: '0655'
      when: ansible_nodename == "stapp01.stratos.xfusioncorp.com"

    - name: Copy story.txt to App Server 2
      ansible.builtin.copy:
        src: /usr/src/data/story.txt
        dest: /opt/data/story.txt
        owner: steve
        group: steve
        mode: '0655'
      when: ansible_nodename == "stapp02.stratos.xfusioncorp.com"

    - name: Copy media.txt to App Server 3
      ansible.builtin.copy:
        src: /usr/src/data/media.txt
        dest: /opt/data/media.txt
        owner: banner
        group: banner
        mode: '0655'
      when: ansible_nodename == "stapp03.stratos.xfusioncorp.com"
```

---

## 🔍 **Check Connectivity**

```
ansible -i /home/thor/ansible/inventory all -m ping
```

<img width="535" height="398" alt="Screenshot 2025-11-28 151647" src="https://github.com/user-attachments/assets/34749c47-c814-4968-b927-dab5712d5f2f" />
<br>

---

## ▶️ **Run the Playbook**

```
ansible-playbook -i /home/thor/ansible/inventory /home/thor/ansible/playbook.yml
```

<img width="955" height="538" alt="Screenshot 2025-11-29 163636" src="https://github.com/user-attachments/assets/10e09ee2-1929-462d-bd83-389fb94b1518" />
<br>

---

## 🔎 **Verify on App Servers**

```
ssh tony@stapp01 ls -l /opt/data/
-rw-r-xr-x 1 tony tony 35 Nov 29 11:05 blog.txt

ssh steve@stapp02 ls -l /opt/data/
-rw-r-xr-x 1 steve steve 27 Nov 29 11:05 story.txt

ssh banner@stapp03 ls -l /opt/data/
-rw-r-xr-x 1 banner banner 22 Nov 29 11:05 media.txt
```  

<img width="619" height="201" alt="Screenshot 2025-11-29 163951" src="https://github.com/user-attachments/assets/37c26a16-f414-4aa2-8090-63d534fdce87" />
<br>

---

## 🎯 Key Learnings

- Ansible **conditionals** enable selective task execution  
- Using **facts** like `ansible_nodename` allows server-specific automation  
- Single playbook can handle multiple server roles cleanly  
- Helps avoid redundant tasks and improves execution efficiency  

---

## 🛠️ Best Practices

- Keep conditional logic **simple and readable**  
- Prefer using **facts** over hard-coded values  
- Structure tasks so each conditional handles a single responsibility  
- Test playbooks with `--check` before applying changes  
- Always use meaningful task names for clarity  

---

## 📘 Official Documentation

- Ansible Conditionals → https://docs.ansible.com/ansible/latest/user_guide/playbooks_conditionals.html
- Ansible Copy Module → https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/copy_module.html

---

## 🚀 Useful tip
If you only want specific facts, you can filter with grep or use setup arguments:

Show only hostname/nodename facts:
```bash
ansible -i ansible/inventory all -m setup -a 'filter=ansible_hostname'
ansible -i ansible/inventory all -m setup -a 'filter=ansible_nodename'
```
---

## 🚀 Final Thoughts

Day 93 really showed how powerful conditional automation can be when managing multi-server deployments.  
A single playbook can handle three different servers intelligently using just a few `when` statements — clean, scalable, and efficient.

---

## 👨‍💻 Author  
**Vamshi Krishna**  
DevOps Engineer | DevOps & Kubernetes Enthusiast  
[Connect on LinkedIn](https://in.linkedin.com/in/vamshi7)  
> ⚙️ _Feel free to fork and contribute — PRs are welcome!_
---
