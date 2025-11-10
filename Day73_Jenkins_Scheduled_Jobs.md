# Day 73 - Jenkins Scheduled Jobs

---

## 🧩 Task Overview  
<br>

<img width="757" height="862" alt="Screenshot 2025-11-08 143821" src="https://github.com/user-attachments/assets/48b044c7-07b7-4710-94e7-e47d263d5101" />  

Create a Jenkins job named **`copy-logs`** that periodically (every 6 minutes) copies Apache logs — both `access_log` and `error_log` — from **App Server 1** to the **Storage Server** at location `/usr/src/dba`.

This demonstrates automating secure log transfer between servers using Jenkins, SSH keys, and cron-based triggers.

---

## ⚙️ Prerequisites
- Jenkins server up and running with admin access.
- SSH access to:
  - App Server 1 (user: `tony`)
  - Storage Server (user: `natasha`)
- Apache web server installed on App Server 1 (log path: `/var/log/httpd/`).
- Jenkins SSH & SSH Agent plugins installed.
- Basic understanding of SSH key-based authentication and Jenkins job configuration.

---

## 🧰 Step‑by‑Step Implementation

### 1️⃣ Generate SSH Key on Jenkins (Jumphost)
```bash
ssh-keygen
```
- Press **Enter** for all prompts to accept defaults (no passphrase needed).  
- Public key will be created at `/home/thor/.ssh/id_rsa.pub`.

### 2️⃣ Copy the Jenkins SSH key to App Server 1
```bash
ssh-copy-id tony@stapp01
```
- Accept the host fingerprint.
- Enter the password for `tony` when prompted.
- Verify connection:
  ```bash
  ssh tony@stapp01
  exit
  ```

### 3️⃣ Generate SSH Key on App Server 1 and Copy to Storage Server
```bash
ssh tony@stapp01
ssh-keygen
ssh-copy-id natasha@ststor01
```
- Accept the host fingerprint.
- Enter `natasha`'s password.
- Verify passwordless connection:
  ```bash
  ssh natasha@ststor01
  exit
  ```

### 4️⃣ Verify Apache Logs on App Server 1
```bash
ls /var/log/httpd/
```
Expected output:
```
access_log  error_log
```

### 5️⃣ Install SSH Plugin in Jenkins
- Navigate to **Manage Jenkins → Plugins → Available plugins**.
- Search for **“SSH”** and **SSH Agent** install.
- Once installed, restart Jenkins (`Restart Jenkins when installation is complete`).

---  
### 🔐 Configure SSH Connection
1. In **Build Environment**, ensure the **SSH Agent** plugin is available.
2. Add credentials using **Manage Jenkins → Credentials → Add Credentials**:
   - Type: **SSH Username with Private Key**
   - ID: `thor-ssh`
   - Username: `tony`
   - Private Key: Use Jenkins’ generated key
3. Save credentials.

---

## 🧠 Jenkins Job Configuration

### 🏗️ Create Job
1. Click **New Item** → Enter name **`copy-logs`** → Select **Freestyle project** → Click **OK**.

### ⚙️ Configure Job Parameters
1. Scroll to **Build Triggers** → Check **“Build periodically”**.
2. Set cron expression:
   ```
   */6 * * * *
   ```
   ➜ This triggers the job every **6 minutes**.
   <br>  
   
   <img width="948" height="442" alt="Screenshot 2025-11-08 144746" src="https://github.com/user-attachments/assets/23181fd6-1216-47e9-b26b-d301f5b18935" />  

### 🧾 Add Build Step
In the **Build** section:
- Click **Add build step → Execute shell (via SSH)** (using SSH plugin).
- Select the SSH Site (previously created credentials)
- Command to execute:
  ```bash
  scp /var/log/httpd/error_log /var/log/httpd/access_log natasha@ststor01:/usr/src/dba/
  ```
  <img width="825" height="492" alt="Screenshot 2025-11-08 144733" src="https://github.com/user-attachments/assets/154f7aa3-1972-4aae-9fa8-1b3c6e9d38a3" />  


### 💾 Save and Apply Configuration  

---

## 🧪 Verify Job Execution

### 🧭 Trigger and Observe Console Output
After 6 minutes (or manual trigger via **Build Now**), view **Console Output**:  

<img width="1261" height="474" alt="Screenshot 2025-11-08 143806" src="https://github.com/user-attachments/assets/e7f6eda5-1537-453d-809e-ae9e00b3ccb6" />  <br>

✅ Both `error_log` and `access_log` are successfully copied to `/usr/src/dba` on the Storage Server.  
✅ Verify by logging into storage server.

---

## 📘 Key Takeaways
- Automating secure file transfer using Jenkins + SSH eliminates manual intervention.  
- SSH key-based authentication is essential for passwordless automation.  
- Jenkins cron syntax (`*/6 * * * *`) ensures timed periodic execution.  
- The **SSH Plugin** simplifies running remote commands directly via Jenkins.  
- Ideal for **centralized log collection**, **backups**, or **monitoring pipelines**.

---

## 🚀 Wrap‑Up
You have successfully:  
✅ Created a periodic Jenkins job (`copy-logs`)  
✅ Configured passwordless SSH between Jenkins, App Server, and Storage Server  
✅ Automated log copy operation using Jenkins SSH commands  
✅ Verified successful transfer and job execution  

---

> 🧠 **Reflection Questions:**  
> 1. How could you extend this job to compress or archive logs before transfer?  
> 2. What security considerations should be made when using SSH credentials in Jenkins?  
> 3. How would you integrate this job into a larger centralized logging or ELK pipeline?  

---
