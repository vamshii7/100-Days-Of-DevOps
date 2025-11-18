# Day 81: Jenkins Multistage Pipeline

## 🎯 Objective  
The xFusionCorp Industries development team is deploying a new static website. Your task is to build a **two‑stage Jenkins Pipeline** that automatically deploys updated code and tests application accessibility via the load balancer.

---

## 📌 Prerequisites  
- Access to Jenkins (admin / Adm!n321)  
- Access to Gitea (sarah / Sarah_pass123)  
- Storage server SSH access (natasha@ststor01)  

---

## 🖥️ Step 1 - Update Code 

- Login to the Gitea  
- Navigate to the repository  
- Update the file:

    ```bash
    index.html to "Welcome to xFusionCorp Industries"
    ```  

- Push the changes to the master branch

---

## 🔐 Step 2 - Validate Storage Directory Permissions  
Since Jenkins will deploy code here, ensure correct permissions:

```bash
ssh natasha@ststor01
sudo chown -R natasha:natasha /var/www/html
ls -lah /var/www/html
```

---

## 🧩 Step 3 - Log in to Jenkins  
Navigate to Jenkins UI → Install required plugin:

### Install  
- **Pipeline Plugin**
- **SSH Credentials Plugin**

(Jenkins ➝ Manage Jenkins ➝ Plugins ➝ Available)

Restart Jenkins if needed.  
Generate SSH key pair and Add SSH Credentials 'storage-server-ssh'.

---

## 🛠️ Step 4 - Create Jenkins Pipeline Job  
Create a new **Pipeline Job** named:

```
deploy-job
```

⚠️ *Do NOT choose Multibranch Pipeline.*

---

## 🚀 Step 5 - Add Two Pipeline Stages  
The task requires:

1. **Deploy Stage** - Pulls the latest code into `/var/www/html` on the Storage Server  
2. **Test Stage** - Verifies the website is working using a curl request on the LB URL  

Use this beginner-friendly Jenkinsfile:

```groovy
pipeline {
    agent any

    stages {
        stage('Deploy') {
            steps {
                sshagent (credentials: ['storage-server-ssh']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no natasha@ststor01 "
                            cd /var/www/html &&
                            git pull origin master
                        "
                    '''
                }
            }
        }

        stage('Test') {
            steps {
                sh '''
                    echo "Testing Application..."
                    curl -I http://stlb01:8091 || exit 1
                '''
            }
        }
    }
}
```

---

## ▶️ Step 6 - Run the Pipeline  
Click **Build Now**  
Monitor the logs from:

```
Build → Console Output
```

Expected output:

- Code deployed  
- curl returns **HTTP/1.1 200 OK**

---

## 🌐 Step 7 - Validate Deployment  
Click the **App** button in your lab environment.

Open:  
```
http://stlb01:8091
```

You should see:

```
Welcome to xFusionCorp Industries
```

Ensure no subdirectory appears (i.e., not `/web` etc.)

---

## ✅ Final Result  
Your Jenkins Multistage Pipeline now:

✔ Deploys the latest code to the storage server  
✔ Automatically tests application availability  
✔ Provides production‑ready CI/CD logic  

---

