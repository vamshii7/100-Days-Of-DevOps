# 🚀 Day 75 - Jenkins Slave Nodes

<img width="1842" height="889" alt="Screenshot 2025-11-10 160840" src="https://github.com/user-attachments/assets/0b6bad40-e622-4f69-85e4-f2313afa7a2e" />


---

## 🚀 **Task Overview**  

<img width="618" height="1033" alt="Screenshot 2025-11-10 153926" src="https://github.com/user-attachments/assets/1fa74a26-e5c2-4948-9c82-87a6213e94be" />  

<br>
The Nautilus DevOps team has installed and configured a new **Jenkins server** in Stratos DC to manage CI/CD and automation tasks. The goal is to **add all application servers as Jenkins slave nodes (build agents)**, allowing distributed build execution and task automation.

---

## 🧠 **Objective**  
Add all app servers as **SSH build agents/slave nodes** in Jenkins, each with its respective label and remote root directory.

### **Server and Label Details**  
| App Server | Node Name     | Label   | Remote Root Directory       |
|-------------|---------------|----------|------------------------------|
| stapp01     | App_server_1  | stapp01 | /home/tony/jenkins          |
| stapp02     | App_server_2  | stapp02 | /home/steve/jenkins         |
| stapp03     | App_server_3  | stapp03 | /home/banner/jenkins        |

---

## 🧩 **Step 1: Generate and Distribute SSH Keys**
First, generate SSH keys on the jumphost to enable passwordless authentication with all app servers.

```bash
ssh-keygen
```
When prompted, press Enter for defaults (no passphrase needed).

Next, copy the SSH key to each app server:

```bash
ssh-copy-id tony@stapp01
ssh-copy-id steve@stapp02
ssh-copy-id banner@stapp03
```

Verify passwordless access:

```bash
ssh tony@stapp01
exit
ssh steve@stapp02
exit
ssh banner@stapp03
exit
```

✅ You should be able to SSH into all servers without entering a password.

---

## ⚙️ **Step 2: Prepare the App Servers**
Ensure **Java** is installed on all app servers since Jenkins agents require it.

```bash
ssh tony@stapp01
sudo yum install -y java-21-openjdk
exit

ssh steve@stapp02
sudo yum install -y java-21-openjdk
exit

ssh banner@stapp03
sudo yum install -y java-21-openjdk
exit
```

---

## 🧩 **Step 3: Install Required Jenkins Plugins**
1. Navigate to **Manage Jenkins → Plugins → Available Plugins**.
2. Search and install the following plugins:
   - **SSH Build Agents**
   - **SSH Credentials Plugin**
   - **SSH Agent Plugin**
3. After installation, click **Restart Jenkins when installation is complete**.

If Jenkins UI freezes, refresh the browser once Jenkins restarts.

---

## 🔐 **Step 4: Add SSH Credentials in Jenkins**
1. Go to **Manage Jenkins → Credentials → Global → Add Credentials**.
2. Select **Kind:** _SSH Username with private key_.
3. Add entries for each app server:
   | Username | Description |
   |-----------|--------------|
   | tony      | stapp01 key |
   | steve     | stapp02 key |
   | banner    | stapp03 key |

5. Paste the private key (`~/.ssh/id_rsa`) generated earlier.

   <img width="875" height="310" alt="Screenshot 2025-11-10 153914" src="https://github.com/user-attachments/assets/13529831-d579-4636-a905-896f344369a8" />  


---

## 🖥️ **Step 5: Add SSH Build Agents (Slave Nodes)**
1. Go to **Manage Jenkins → Nodes → New Node**.
2. For each app server, configure as follows:

### **App_server_1 (stapp01)**
- Node Name: `App_server_1`
- Remote Root Directory: `/home/tony/jenkins`
- Labels: `stapp01`
- Launch Method: _Launch agents via SSH_
- Host: `stapp01`
- Credentials: `tony`
- Host Key Verification Strategy: _Non verifying verification strategy_

  <img width="1103" height="1288" alt="Screenshot 2025-11-10 155204" src="https://github.com/user-attachments/assets/abf6f9c8-ffc3-489f-b35f-71cfa938a1cb" />  


Repeat the same for:

| Node Name     | Remote Directory        | Label   | Host     | Credential |
|----------------|--------------------------|----------|-----------|-------------|
| App_server_2  | /home/steve/jenkins      | stapp02 | stapp02  | steve       |
| App_server_3  | /home/banner/jenkins     | stapp03 | stapp03  | banner      |

✅ Click **Save** and verify each node shows **Online** status.  

<img width="1261" height="508" alt="Screenshot 2025-11-10 155609" src="https://github.com/user-attachments/assets/943981e8-9d71-48f5-9887-d600a580ae67" />  


---

## 🧪 **Step 6: Verify Nodes with a Test Job**  
1. Create a new Jenkins job named **Test-Job**.  
2. Under **General → Restrict where this project can be run**, enter a label (e.g., `stapp01`).  

   <img width="1210" height="347" alt="Screenshot 2025-11-10 160107" src="https://github.com/user-attachments/assets/0ee86b25-23ff-40ab-9a93-a3d441af64f5" />  

3. Add a build step:
   ```bash
   hostname
   ```
   <img width="863" height="430" alt="Screenshot 2025-11-10 160234" src="https://github.com/user-attachments/assets/801a5834-72a2-432f-ab9a-a47401cdb05e" />  

4. Click **Save** → **Build Now**.  
5. Check **Console Output** — you should see the hostname and workspace path of the respective app server.

   <img width="1255" height="345" alt="Screenshot 2025-11-10 160401" src="https://github.com/user-attachments/assets/3492ef0a-5648-488b-830e-ebd8c2ca12da" />  


Repeat the same test for `stapp02` and `stapp03` by changing labels.

---

## ✅ **Verification**
All Jenkins slave nodes (App_server_1, App_server_2, App_server_3) should be **Online** and capable of running build jobs via SSH.

Example console output:  

  <img width="1255" height="345" alt="Screenshot 2025-11-10 160401" src="https://github.com/user-attachments/assets/3492ef0a-5648-488b-830e-ebd8c2ca12da" /> 

---

## 💡 **Final Thoughts & Learnings**
- Jenkins slave nodes distribute the build load efficiently across servers.
- SSH-based agent setup is secure and flexible for automation.
- Always ensure Java is installed on agents before connecting.
- Using labels helps route specific jobs to targeted environments.

🎯 **End Result:** Successfully onboarded all app servers as Jenkins SSH build agents.

---

**🏁 Congratulations!** You’ve completed **Day 75 - Jenkins Slave Nodes** of your *100 Days of DevOps* journey.

