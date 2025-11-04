# Day 68 - Set Up Jenkins Server

Jenkins is a leading open-source automation server used for **Continuous Integration (CI)** and **Continuous Delivery (CD)**. It automates building, testing, and deploying applications, making it a cornerstone of modern DevOps practices.  
In this task, we’ll set up a Jenkins server from scratch, configure it to run as a service, and secure initial access.  <br>

<img width="613" height="564" alt="Screenshot 2025-11-03 140303" src="https://github.com/user-attachments/assets/2a6d022e-e562-42df-a51e-7f801c091805" />  <br>


---

## 📌 Objectives  
- Install Jenkins on a Linux server using `yum`.  
- Enable and start Jenkins as a systemd service.   
- Retrieve the initial admin password and create the first admin user.  

---
## 🛠️ Step-by-Step Setup  

Login to Jenkins Server with the provided credentials  

### 1. Add Jenkins Repository  
```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo \
  https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
```
### 2. Update Packages  
```bash
sudo yum upgrade -y
```
### 3. Install Dependencies  
Jenkins requires Java. Install OpenJDK and fontconfig:  
```bash
sudo yum install fontconfig java-21-openjdk -y
```
### 4. Install Jenkins  
```bash
sudo yum install jenkins -y
```
### 5. Enable and Start Jenkins Service
```bash
sudo systemctl daemon-reload
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
```  
  - `enable` ensures Jenkins starts on boot.  
  - `status` confirms Jenkins is running.  <br>

<img width="1186" height="287" alt="image" src="https://github.com/user-attachments/assets/c011fe8c-383f-4d36-a6bb-67553b10c064" />  <br>


### 6. Access Jenkins  
- By default, Jenkins listens on port 8080. Open a browser and go to:
```bash
http://<ServerIP>:8080
```
### 7. Unlock Jenkins  
Retrieve the initial admin password:  
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
Paste this password into the Jenkins setup wizard to continue.  

### 8. Complete Setup Wizard  

- Install suggested plugins (recommended for beginners).  
- Create the first admin user (example):  
  - Username: theadmin  
  - Password: Adm!n123  
  - Email: john@jenkins.stratos.xfusioncorp.com  <br>

<img width="1016" height="1297" alt="Screenshot 2025-11-03 140253" src="https://github.com/user-attachments/assets/b24533f4-c6e2-47a2-b0c1-8c610cdc2296" />  <br>
<img width="2559" height="644" alt="Screenshot 2025-11-03 140405" src="https://github.com/user-attachments/assets/2c7c0e69-f572-4657-bfd9-5299ae9dd8da" />  <br>

---  

## 🧭 Best Practices  
- Run Jenkins under its own user (default) — not as root.  
- Use HTTPS in production with a reverse proxy (Nginx/Apache).  
- Backup Jenkins home (/var/lib/jenkins) regularly.  
- Install only required plugins to reduce maintenance overhead.  
- Integrate with SCM (GitHub/GitLab/Bitbucket) for automated builds.

---  

## 💡 Key Learnings  
- Jenkins installation on RHEL/CentOS is straightforward using the official repo.
- The systemd service ensures Jenkins runs reliably and restarts on reboot.
- The initial admin password is critical for first-time setup.
- Jenkins’ strength lies in its plugin ecosystem and integrations.  

---  

## 🎯 Outcome  
- Successfully set up a Jenkins server with:
- Installed and running Jenkins service.
- Initial admin user created and dashboard accessible.
- This lays the foundation for building CI/CD pipelines in upcoming tasks.
 ---  
