# 🚀 Day 77: Jenkins Deploy Pipeline

> *"Automation isn’t just about speed — it’s about consistency, reliability, and empowering teams to deliver seamlessly."*

Today’s challenge focuses on creating a **Jenkins Pipeline job** to deploy a static website to Nautilus App Servers.  <br>  

<img width="734" height="799" alt="Screenshot 2025-11-12 142444" src="https://github.com/user-attachments/assets/ab4bd048-91c0-4e1c-bfe4-963a8e72424a" />


We’ll configure a **Jenkins slave node**, connect it to the master, and build a pipeline that pulls the latest code from a Git repository.  

---


## 🔑 Step 1: Install Pipeline Plugin

1. Log in to Jenkins UI as `admin`.
2. Navigate to **Manage Jenkins → Manage Plugins**.
3. Search for and install **Pipeline Plugin**.
4. Restart Jenkins once the installation completes.

---

## ⚙️ Step 2: Prepare the Storage Server

1. SSH into the Storage Server:
   ```bash
   ssh user@storageserver
   ```

2. Verify the local Git repository:
   ```bash
   ls -lah /var/www/html
   ```

3. Install Java runtime:
   ```bash
   sudo yum install -y java-21-openjdk
   ```  
---

## 🛠 Step 3: Add Jenkins Slave Node

1. Go to **Manage Jenkins → Nodes → New Node**.
2. Configure the node as follows:

   | Setting | Value |
   |----------|--------|
   | **Node Name** | `Storage Server` |
   | **Remote Root Directory** | `/var/www/html` |
   | **Labels** | `ststor01` |
   | **Launch Method** | Launch agent by connecting it to the controller |  

   <img width="760" height="494" alt="Screenshot 2025-11-12 144007" src="https://github.com/user-attachments/assets/21607c39-218a-439d-9611-dffbfcbb39d4" />

4. Click **Save** to add the node.

---

## 🔗 Step 4: Connect Slave to Jenkins Controller

Go to Manage Jenkins > System and check jenkins location url, it should be http://jenkins:8080/ (check in users and servers details).

<img width="624" height="262" alt="Screenshot 2025-11-12 145220" src="https://github.com/user-attachments/assets/d3801999-b516-4d2d-a9d7-1282f2adb097" />  <br>  

### Click on Storage Server in the Nodes > Under "run from agent command line, with the secret stored in a file: (Unix)"

On the Storage Server, execute the following commands: 

```bash
echo f729c2135e395b40daf4ad97469de0ebbd023cd782edaa0ca14c21fdc12bb1ff > secret-file # This might vary for you
curl -sO http://jenkins:8080/jnlpJars/agent.jar
sudo java -jar agent.jar -url http://jenkins:8080/ -secret @secret-file -name "Storage Server" -webSocket -workDir "/var/www/html"
```  

<img width="1188" height="197" alt="Screenshot 2025-11-12 144115" src="https://github.com/user-attachments/assets/b1958691-bff6-4eee-8d60-0a41c636156a" />  <br>  
<img width="518" height="86" alt="Screenshot 2025-11-12 145536" src="https://github.com/user-attachments/assets/b47b0dda-fd63-4240-9d73-3aac70181074" />  <br>  
<img width="1140" height="326" alt="Screenshot 2025-11-12 145823" src="https://github.com/user-attachments/assets/e2a86f99-9833-49fa-a27e-152d0e23ee3c" />  <br>  

✅ Once executed, the slave should appear **online and connected** in the Jenkins UI.  

<img width="1354" height="388" alt="Screenshot 2025-11-12 145929" src="https://github.com/user-attachments/assets/81b7a3d4-176c-4d3c-9739-27fbc36f5e05" />


---

## 📦 Step 5: Create Jenkins Pipeline Job

1. In Jenkins UI, click **New Item** → name it `nautilus-webapp-job`.
2. Select **Pipeline** (not Multibranch).

   <img width="965" height="738" alt="Screenshot 2025-11-12 150018" src="https://github.com/user-attachments/assets/6a01653b-c24a-4261-86fc-19f602b11adf" />  

3. Under the **Pipeline** section, add the following script:

   ```groovy
   pipeline {
       agent { 
           label 'ststor01' 
       }

       stages {
           stage('Deploy') {
               steps {
                   echo '🚀 Starting Deployment for web_app'
                   sh 'cd /var/www/html && git pull'
                   echo '✅ Completed Deployment for web_app'
               }
           }
       }
   }
   ```

4. Save the configuration and click **Build Now**.

---

## 🖥️ Console Output Example

```
Started by user admin
[Pipeline] Start of Pipeline
[Pipeline] node
Running on Storage Server in /var/www/html/workspace/nautilus-webapp-job
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Deploy)
[Pipeline] echo
🚀 Starting Deployment for web_app
[Pipeline] sh
+ cd /var/www/html
+ git pull
Already up to date.
[Pipeline] echo
✅ Completed Deployment for web_app
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
Finished: SUCCESS
```

---

## ✅ Summary

| Step | Action |
|------|--------|
| **Install Plugin** | Pipeline Plugin via Jenkins UI |
| **Prepare Server** | Verify repo, install Java |
| **Add Node** | Add Storage Server with label `ststor01` |
| **Connect Agent** | Run `agent.jar` with secret file |
| **Pipeline Job** | Deploy stage executes `git pull` |

---

## 💡 Best Practices & Key Learnings

- **Least Privilege Principle:** Configure nodes with only the access they need.  
- **Pipeline as Code:** Use Groovy scripts for versioned, reproducible deployments.  
- **Agent Connectivity:** Always confirm that slave nodes are online before triggering jobs.  
- **Console Validation:** Check Jenkins logs for successful deployment confirmation.  

---

## 🎯 Final Thoughts

This exercise demonstrates how **Jenkins Pipelines** can automate deployments across distributed environments.  
By connecting the Storage Server as a Jenkins agent and orchestrating a simple `git pull`, we ensure the Nautilus App Servers always serve the latest version of the static site.

> 💬 *"A pipeline is more than automation — it’s the heartbeat of continuous delivery."* 💪
