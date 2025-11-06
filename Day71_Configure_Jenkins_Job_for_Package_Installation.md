# 🚀 Day 71: Configure Jenkins Job for Package Installation  


This challenge focuses on automating **package installation across servers** using Jenkins.  
We’ll create a **parameterized Freestyle job** that connects to a remote storage server via SSH and installs packages dynamically based on user input.  

By the end, you’ll have a secure and reusable Jenkins automation for remote package deployment, a real-world DevOps use case.  

---

## 🧩 Challenge Overview

The Nautilus DevOps team needed to automate the installation of application packages on their **storage server (ststor01)** from a Jenkins controller running in the Stratos Datacenter.

### ✅ Requirements
- Create a **Jenkins Freestyle job** named `install-packages`
- Add a **string parameter** named `PACKAGE`
- Configure the job to:
  - Connect to the **storage server** via SSH (`natasha@ststor01`)
  - Install the package specified in `$PACKAGE`
- Ensure the job can run successfully multiple times without manual intervention  

---

## ⚙️ Step 1: Install Required Plugins

From Jenkins Dashboard:  
- Navigate to **Manage Jenkins → Manage Plugins → Available**
- Search and install:
  - ✅ **Publish Over SSH & SSH** — allows remote command execution via SSH  
  - ✅ *(Optional)* **SSH Agent** — useful for injecting SSH credentials into freestyle or pipeline jobs
- After installation, choose  
  **Restart Jenkins when installation is complete and no jobs are running**
- Wait for Jenkins to restart and re-login as `admin (Adm!n321)`

---

## 🔐 Step 2: Configure SSH Access  

We’ll set up **passwordless SSH authentication** so Jenkins can connect to the remote server (`ststor01`) as `natasha`.

### 1️⃣ Generate an SSH Key  
On the Jenkins server terminal:  
```bash

ssh-keygen -t ed25519 -C "natasha@ststor01" -f /var/lib/jenkins/.ssh/id_ed25519 -N ""
```

### 2️⃣ Copy Public Key to the Remote Storage Server
```bash
ssh-copy-id -i /var/lib/jenkins/.ssh/id_ed25519.pub natasha@ststor01
```

### 3️⃣ Test the Connection
```bash
ssh natasha@ststor01 "hostname"
```
✅ If you get the hostname without a password prompt, the SSH key works.

---

## 🧰 Step 3: Add Global SSH Credentials in Jenkins  

1. Go to **Manage Jenkins → Credentials → System → Global credentials (unrestricted) → Add Credentials**  
2. Choose:
   - Kind: **SSH Username with private key**
   - ID: `storage-ssh`
   - Username: `natasha`
   - Private Key: **Enter directly** → paste the contents of  
     `/var/lib/jenkins/.ssh/id_ed25519`
   - Description: `SSH key for natasha@ststor01`
3. Click **Create**  

<img width="1017" height="1212" alt="Screenshot 2025-11-06 151017" src="https://github.com/user-attachments/assets/181b7242-0038-43e7-a2ef-0b73bec35242" />
<br>

---

## 🌐 Step 4: Configure SSH Server in Jenkins  

1. Navigate to **Manage Jenkins → Configure System**
2. Scroll to the **Publish over SSH** section
3. Click **Add** under SSH Servers and fill:
   - Name: `storage-server`
   - Hostname: `ststor01`
   - Username: `natasha`
4. Click **Test Configuration**  
✅ You should see *“Success”*
5. Click **Save**

<img width="1027" height="1189" alt="Screenshot 2025-11-06 151933" src="https://github.com/user-attachments/assets/a4aa6915-2a52-438c-ab93-6bd3f3a5c742" />  
<br>

---

## 🌐 Step 5: Configure SSH Server in Jenkins  

1. Scroll to the **SSH remote hosts** section
2. Click **Add** under SSH Servers and fill:  
   - Hostname: `ststor01`
   - Credentials: `natasha`
3. Click **Save**  
<img width="1017" height="740" alt="Screenshot 2025-11-06 151950" src="https://github.com/user-attachments/assets/24e7ef60-56f8-4dfc-bfff-6df2367578c2" />


---

## 🧩 Step 6: Create the Freestyle Job

### **Job Name:** `install-packages`

1. Go to **Dashboard → New Item → Freestyle project → OK**
2. Under **General**, check **This project is parameterized**
3. Add **String Parameter**:
   - Name: `PACKAGE`
   - Default value: `httpd`
   - Description: `Enter the package name to install on storage server`  

<img width="661" height="467" alt="Screenshot 2025-11-06 152609" src="https://github.com/user-attachments/assets/0f5edb8c-4b11-4270-b1d0-7ec8e483a01c" />
<br>
<img width="1189" height="764" alt="Screenshot 2025-11-06 125711" src="https://github.com/user-attachments/assets/2c26e25a-fb02-48b8-8eaf-7531b11ed92f" />
<br>

---

## 🧱 Step 7: Add Build Step to Execute Commands Over SSH  

1. Scroll to **Build → Add build step → Execute shell script on remote host using SSH**
2. Configure as:
   - **SSH Site:** `natasha@stsor01:22`
   - **Command:**
     ```bash
      sudo yum install -y $PACKAGE
     ```
3. Click **Save**

<img width="793" height="539" alt="Screenshot 2025-11-06 153049" src="https://github.com/user-attachments/assets/1c65d28c-6775-4645-bc94-e2011dd29154" />  

---

## 🧪 Step 8: Test the Job

1. Click **Build with Parameters**
2. Enter:
   ```bash
   PACKAGE=httpd
   ```
   <img width="1148" height="393" alt="Screenshot 2025-11-06 153107" src="https://github.com/user-attachments/assets/da25ffdb-b0c3-4be9-ad9e-004482e58202" />
   <br>
3. Observe the **Console Output**:  
   <img width="952" height="467" alt="Screenshot 2025-11-06 153318" src="https://github.com/user-attachments/assets/319fd698-3d69-4855-9cf3-9d4cdeab3807" />
   <br>

---

## 🧰 Step 9: Troubleshooting Tips

| Issue | Resolution |
|--------|-------------|
| **Permission denied (publickey)** | Ensure Jenkins’ SSH key is added to `natasha@ststor01` |
| **sudo: a terminal is required** | Add the below line to `/etc/sudoers`: <br>`natasha ALL=(ALL) NOPASSWD: /usr/bin/apt, /usr/bin/yum` |
| **Host key verification failed** | Add `-o StrictHostKeyChecking=no` to your SSH command or accept the key manually |
| **Job succeeds but package not installed** | Check whether you’re running with correct sudo privileges or the package name is valid |

---

## 🧠 Key Learnings & Best Practices

- Always use **SSH keys** for secure, passwordless automation.  
- Use **Publish Over SSH** for lightweight remote execution.  
- Implement **parameterized jobs** for flexible and reusable automation.  
- Enforce **NOPASSWD sudo** only for specific commands to maintain security.  
- Keep logs clear and include success/failure states in output.  

---

## 🏁 Final Thoughts  

This task demonstrates how to integrate Jenkins with remote infrastructure for **secure, parameterized automation**.  
By mastering SSH-based deployments and Freestyle jobs, you can extend Jenkins beyond CI/CD — using it as a **remote orchestration tool** for configuration management and server provisioning.

---
