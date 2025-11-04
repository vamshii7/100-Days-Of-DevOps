# Day 69: Install Jenkins Plugins

Jenkins’ true power lies in its **plugin ecosystem**. Plugins extend Jenkins’ core functionality, enabling integrations with version control systems, build tools, cloud providers, and more. In this task, we’ll install essential plugins like **Git** and **GitLab**, which are commonly used in CI/CD pipelines.

---  
<br> 

<img width="603" height="612" alt="Screenshot 2025-11-04 103759" src="https://github.com/user-attachments/assets/a8f96999-ed06-4326-9031-305d36dcb833" />  


## 📌 Objectives
- Log in to Jenkins with admin credentials.
- Install the **Git** and **GitLab** plugins.
- Restart Jenkins if required to complete installation.
- Verify plugins are installed and available for use in jobs.

---

## 🛠️ Step-by-Step Setup

### 1. Log in to Jenkins
- Navigate to your Jenkins instance: `http://<ServerIP>:8080`
- Log in with the admin credentials provided.

---

### 2. Access Plugin Manager
- From the Jenkins dashboard, go to: `Manage Jenkins → Plugins`
- Use the **Available** or **Installed** tabs to search for plugins.  

---

### 3. Install Git Plugin
- Search for **Git plugin**.  
- Select the latest stable version and click **Install without restart**.

---  

### 4. Install GitLab Plugin
- Search for **GitLab plugin**.  
- Select and install it.  
- If prompted, choose **Restart Jenkins when installation is complete and no jobs are running**.

---

### 5. Verify Installation
- Go to: `Manage Jenkins → Plugins → Installed`
- Confirm that:
- **Git plugin** is listed (provides Git SCM integration).  
- **GitLab plugin** is listed (enables GitLab triggers and integration).

<br>

<img width="1065" height="590" alt="Screenshot 2025-11-04 110104" src="https://github.com/user-attachments/assets/689db31b-e983-47f0-b1b0-56a5ef42e041" />  


---

## ✅ Validation
- Create a new Jenkins job:
- Under **Source Code Management**, confirm **Git** is available.  
- Under **Build Triggers**, confirm **GitLab** options are available.  
- This ensures both plugins are active and ready for use.

---

## 🧭 Best Practices
- Always install plugins from the **official Jenkins update center**.  
- Avoid installing unnecessary plugins to reduce attack surface and maintenance overhead.  
- Keep plugins updated to patch vulnerabilities.  
- Test plugin compatibility in a staging Jenkins before applying to production.  

---

## 💡 Key Learnings
- Jenkins plugins are the backbone of its extensibility.  
- Git plugin enables Jenkins to pull code from repositories.  
- GitLab plugin allows GitLab to trigger Jenkins builds and report results.  
- Restarting Jenkins after plugin installation ensures stability.  

---

## 🎯 Outcome
Successfully installed and verified **Git** and **GitLab** plugins in Jenkins.  
This enables Jenkins to integrate with Git-based workflows and GitLab CI/CD triggers, setting the stage for building automated pipelines in upcoming tasks.  

<img width="1839" height="960" alt="Screenshot 2025-11-04 110227" src="https://github.com/user-attachments/assets/01b7a938-54ea-46ad-9294-f103b8ca19e8" />


---


