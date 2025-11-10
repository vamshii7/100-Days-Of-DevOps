# 🚀 Day 74 - Jenkins Scheduled Jobs Automation  
<img width="613" height="1015" alt="Screenshot 2025-11-09 144202" src="https://github.com/user-attachments/assets/4b6124f9-82fa-4efd-ad84-f98a0aa348cc" />  

## 🧩 Task Objective  
Create a Jenkins job that:  
- Takes a MySQL database dump from the database server (kodekloud_db01).
- Transfers the dump to a backup server (stdbkp01).
- Schedules the job to run automatically every 10 minutes.

This demonstrates automating secure log transfer between servers using Jenkins, SSH keys, and cron-based triggers.  
---  
## ⚙️ Prerequisites
- Jenkins server up and running with admin access.
- SSH access to:
  - Database Server (user: `peter`)
  - Backup Server (user: `clint`)
- Jenkins SSH & SSH Agent plugin installed.
- Basic understanding of SSH key-based authentication and Jenkins job configuration.

---

## 🧰 Step‑by‑Step Implementation

### Generate SSH Key on Jumphost
```bash
ssh-keygen
```
- Press **Enter** for all prompts to accept defaults (no passphrase needed).  
- SSH Key will be created at `/home/thor/.ssh/id_rsa`.

### Copy the SSH key to Database Server
```bash
ssh-copy-id peter@stdb01
```
- Accept the host fingerprint.
- Enter the password for `peter` when prompted.
- Verify connection:
  ```bash
  ssh peter@stdb01
  exit
  ```

### Generate SSH Key on Database Server and Copy to Backup Server
```bash
ssh peter@stdb01
ssh-keygen
ssh-copy-id clint@stbkp01
```
- Accept the host fingerprint.
- Enter `clint`'s password.
- Verify passwordless connection:
  ```bash
  ssh clint@stbkp01
  exit
  ```

## 🪪 Step 1: Login to Jenkins & Add Credentials

1. Open Jenkins Dashboard → **Manage Jenkins → Credentials → Global → Add Credentials**  
2. Select **SSH Username with Private Key** 
3. Add credentials
   - ID: `peter-ssh`
   - Username: `peter`
   - Private Key: Use generated key
4. Click **Save**  

---

## 🔌 Step 2: Install Required Plugins & Restart  

- Navigate to **Manage Jenkins → Plugins → Available plugins**.
- Search for **“SSH”** and **“SSH Agent”** and install.
- Once installed, restart Jenkins (`Restart Jenkins when installation is complete`).  
---

## 🏗️ Step 3: Create a Freestyle Job  

1. Click **New Item → Freestyle Project → “database-backup”**
2. Scroll to **Build Triggers** → Check **“Build periodically”**.  
   Set cron expression:
   ```
   */10 * * * *
   ```
   <img width="1164" height="380" alt="Screenshot 2025-11-09 144324" src="https://github.com/user-attachments/assets/56216092-bc73-488e-ada4-36d2ad15762e" />  
   ➜ This triggers the job every **10 minutes**.  
4. Under Environment enable SSH Agent and choose peter's credentials.
   
   <img width="878" height="454" alt="Screenshot 2025-11-09 150146" src="https://github.com/user-attachments/assets/4749e8f9-42ed-4ef2-806d-f9b642411a18" />  
6. Add Build Step
```bash
FILE_PATH="/tmp/db_backups/db_$(date +%F).sql"
ssh -o StrictHostKeyChecking=no peter@stdb01 "mysqldump -u kodekloud_roy -pasdfgdsd kodekloud_db01 > '${FILE_PATH}' && \
scp -o StrictHostKeyChecking=no '${FILE_PATH}' clint@stbkp01:/home/clint/db_backups/"
```  
<img width="1177" height="444" alt="image" src="https://github.com/user-attachments/assets/0c47bd03-3638-4e29-afb1-426ca9bd0db7" />  

> 💡 Use parameters for database name, database user & password

✅ This ensures:
- The dump is created on the remote host.  
- The scp is executed from the remote host, where the file exists.  

---  

## 📜 Step 6: Console Output & Verification

<img width="1583" height="728" alt="Screenshot 2025-11-09 232817" src="https://github.com/user-attachments/assets/fa09abd9-f326-444e-9d1e-b55a211ae619" />  
<img width="581" height="227" alt="Screenshot 2025-11-09 232929" src="https://github.com/user-attachments/assets/acc4f2ad-6a73-43d4-833c-0cff584aee16" />

---

## 🧠 Final Thoughts & Learnings

> 🔹 Learned how to schedule Jenkins jobs efficiently using **cron expressions**  
> 🔹 Improved knowledge of **credentials management** and plugin usage  
> 🔹 Automated database backups with dynamic naming for the dump file  
> 🔹 Strengthened CI/CD automation skills by handling repetitive maintenance tasks  

---
