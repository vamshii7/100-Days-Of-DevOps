# 🚀 Day 79: Jenkins Deployment Job  

## 🧭 Overview  

The Nautilus DevOps team wants **automatic deployment** whenever developers push code to the **master** branch of the repository hosted on Gitea.

<img width="843" height="1230" alt="Screenshot 2025-11-14 144320" src="https://github.com/user-attachments/assets/85dea163-75f9-4bad-bc31-08e587a8791d" />  

---

## 🛠 Step-by-Step Implementation  

# A. Install & Configure HTTPD on All App Servers  

SSH into each app server:

### 1. Install Apache
```bash
sudo yum install -y httpd
```

### 2. Change Default Port to 8080
```bash
sudo sed -i 's/^Listen 80/Listen 8080/' /etc/httpd/conf/httpd.conf
```

If VirtualHost exists:
```bash
sudo sed -i 's/<VirtualHost \*:80>/<VirtualHost *:8080>/' /etc/httpd/conf.d/*.conf 2>/dev/null
```

### 3. Start & Enable Service
```bash
sudo systemctl enable --now httpd
```

### 4. Validate
```bash
curl localhost:8080
```  
<br>

  <img width="826" height="103" alt="Screenshot 2025-11-14 144406" src="https://github.com/user-attachments/assets/b04e544f-4d7f-4699-a351-360341816b1e" />


---

# B. Prepare Storage Server  

SSH into Storage Server as `natasha` or another sudoer.

### 1. Ensure Webroot Exists  
```bash
ls -ld /var/www/html
```

### 2. Set Ownership  
```bash
sudo chown -R sarah:sarah /var/www/html
```

### 3. Verify Sarah's Repo  
```bash
sudo -u sarah ls -la /home/sarah/web
```

  <img width="428" height="110" alt="Screenshot 2025-11-14 144944" src="https://github.com/user-attachments/assets/70f8d693-5704-4de1-8386-7b2929b78f13" />  

  <img width="517" height="356" alt="Screenshot 2025-11-14 144905" src="https://github.com/user-attachments/assets/a5caae69-2a17-4773-9771-4e15c50778e1" />  
  


---

# C. Jenkins Setup  

### 1. Login  

  Login to Jenkins UI 

### 2. Set Jenkins URL  
**Manage Jenkins → Configure System → Jenkins URL**  
Set:  
```
http://jenkins:8080/
```

### 3. Install Required Plugins  
**Manage Jenkins → Manage Plugins → Available**  

Install:  
- Git Plugin  
- Credentials Plugin  
  
Restart when prompted.  

### 4. Add Storage server as a slave  
  
  [Refer Day 77](https://github.com/vamshii7/100-Days-Of-DevOps/blob/main/Day77_Jenkins_Deploy_Pipeline.md)

---

# D. Jenkins Credentials  

Navigate:  
**Manage Jenkins → Manage Credentials → Global → Add Credentials**  

Add Gitea credentials:
- Type: **Username with password**  
- Username: `sarah`  
- Password: `Sarah_pass123`  
- ID: `gitea-sarah`

  <img width="1162" height="773" alt="Screenshot 2025-11-14 145510" src="https://github.com/user-attachments/assets/a67c1427-c522-4ff2-a2ba-0ced08a25768" />


---

# E. Create Jenkins Job: `nautilus-app-deployment`  

**New Item → Freestyle Project → nautilus-app-deployment**  

### 1. General  
✔ Tie job to Storage Server node (label: `storage`)  

### 2. Source Code Management  
- Select **Git**  
- Repository URL:
```
https://<YOUR-LBR-URL>/sarah/web
```
- Credentials: **gitea-sarah**  
- Branch:
```
*/master
```
  <img width="1211" height="962" alt="Screenshot 2025-11-14 150649" src="https://github.com/user-attachments/assets/6610839f-55a8-40b7-b8d0-9e6f309d328e" />


### 3. Build Triggers  
If webhooks available:  
✔ *Build when a change is pushed to Git repository*  

Otherwise use SCM Polling:
```
* * * * *
```

### 4. Build Steps → Execute Shell  
```bash
cd /var/www/html

# Mark directory as safe for git
git config --global --add safe.directory /var/www/html

# Pull latest code
git pull

```

This deploys the entire repository content.

---

# F. Test Deployment Trigger (Git Push)  

SSH as sarah:
```bash
ssh sarah@storage-server
cd ~/web
```

Modify file:
```bash
echo "Welcome to the xFusionCorp Industries" > index.html
```

Commit & push:
```bash
git add index.html
git commit -m "updated index.html"
git push origin master
```

Jenkins will be triggered automatically.

---

## 📊 Jenkins Console Output  

<img width="1281" height="688" alt="Screenshot 2025-11-14 151807" src="https://github.com/user-attachments/assets/528f9f6f-8cb8-478b-98d4-9e2b2a7eb063" />  

---

## 🌐 Verification  

### On Storage Server:
```bash
ls -la /var/www/html
cat /var/www/html/index.html
```

### Checking Git log:  
```bash
cd /var/www/html
git log
```

<img width="626" height="252" alt="Screenshot 2025-11-14 152021" src="https://github.com/user-attachments/assets/9049f028-55dd-46e7-a27b-0242de56901e" />  
<img width="1163" height="457" alt="Screenshot 2025-11-14 151922" src="https://github.com/user-attachments/assets/10c43c81-255a-42f8-a3ca-3862452c46a8" />  



### On App Servers:
```bash
curl http://localhost:8080
```  

<img width="732" height="163" alt="Screenshot 2025-11-14 151932" src="https://github.com/user-attachments/assets/6303c8a2-bd39-4a01-aed9-d40bd927ebc0" />  

---

## 🐞 Troubleshooting  

### 1. Permission Denied in Jenkins  
```bash
sudo chown -R sarah:sarah /var/www/html
```

### 2. Jenkins Cannot Run Sudo  
Edit:
```
/etc/sudoers.d/jenkins
```
Add:
```
jenkins ALL=(ALL) NOPASSWD: /usr/bin/rsync, /bin/chown, /usr/bin/git
```

### 3. Webhook Not Triggering  
Use **Poll SCM** as fallback.

---

## ✔ Validation Checklist  

| Task | Status |
|------|--------|
| httpd installed on all app servers | ✔ |
| httpd running on 8080 | ✔ |
| /var/www/html owned by sarah | ✔ |
| Jenkins job created | ✔ |
| SCM webhook / polling working | ✔ |
| Push triggers deployment | ✔ |
| Browser shows updated content | ✔ |

---

## 📎 Useful Commands  

Check Apache port:
```bash
ss -lntp | grep httpd
```

Jenkins logs:
```bash
sudo journalctl -u jenkins -f
```

Git logs:
```bash
git log --oneline -5
```

---

🎉 **End of Day 79 – Jenkins Deployment Job**  

## 🏆 Final Thoughts

Day 79 brings together CI, CD, Git workflows, node-based deployments, security best practices, and automated web server updates.  
This task reflects real-world DevOps responsibilities - especially in distributed environments with shared storage and multi-server deployments.  

---
