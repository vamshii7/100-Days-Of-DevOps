# 🚀 Jenkins Pipeline: Build & Push Nginx Docker Image

## 📝 Task Overview

<img width="695" height="541" alt="Screenshot 2025-11-23 140127" src="https://github.com/user-attachments/assets/df15e592-947c-4e42-a059-06746fb32d21" />  

<br>  

- Build a Docker image using Jenkins Pipeline.
- Use the Dockerfile from the Gitea repo `sarah/web`.
- Push the image to the registry `stregi01.stratos.xfusioncorp.com:5000`.
- Run the pipeline on Jenkins agent **App Server 2** (`stapp02`).
- Use a single pipeline stage named **Build**.

---

## ⚙️ Setup Steps

### 🔌 Install Jenkins Plugins
Ensure the following plugins are installed and Jenkins is restarted:
- **Pipeline**
- **Git**
- **Credentials**

### 🔐 Add Git Credentials
- Go to **Manage Jenkins → Credentials**.
- Add a credential with ID `sarah-git` for accessing Gitea (`http://git.stratos.xfusioncorp.com/sarah/web.git`).

### 🖥️ Add Jenkins Agent
- Add a node labeled `stapp02` (App Server 2). [Check Day77](https://github.com/vamshii7/100-Days-Of-DevOps/blob/main/Day77_Jenkins_Deploy_Pipeline.md)
- Confirm it’s online and available for builds.

---

## 📦 Jenkins Pipeline Configuration

Create a pipeline job named `nginx-container`. 
Check your repo URL and branch name.
Use the following script:

```groovy
pipeline {
    agent { label 'stapp02' }

    stages {
        stage('Build') {
            steps {
                // Checkout from Gitea
                git branch: 'master',
                    url: 'http://git.stratos.xfusioncorp.com/sarah/web.git',
                    credentialsId: 'sarah-git'

                // Build & push Docker image
                sh """
                git pull origin master
                docker build -t stregi01.stratos.xfusioncorp.com:5000/nginx:latest .
                docker push stregi01.stratos.xfusioncorp.com:5000/nginx:latest
                """
            }
        }
    }
}
```

---

## 📊 Console Output Highlights

- **Git Checkout:** Successfully fetched and checked out master branch from Gitea.
- **Docker Build:** Used nginx:stable-alpine3.17-slim as base image.
- **Image Push:** Pushed to registry stregi01.stratos.xfusioncorp.com:5000/nginx:latest.

  ```bash
  Started by user admin
  Running on App_Server_2
  Checking out Revision 52b85d0...
  Commit message: "Added dockerfile"
  + docker build -t stregi01.stratos.xfusioncorp.com:5000/nginx:latest .
  + docker push stregi01.stratos.xfusioncorp.com:5000/nginx:latest
  latest: digest: sha256:4413305a1d18c7a3a6c6eb82ac1a11aa8139cda74ad1629e32788c501c3989c4
  Finished: SUCCESS
  ```
---

## ✅ Outcome

- Docker image built and pushed successfully.
- Pipeline executed on App Server 2 (stapp02).
- Gitea integration and registry authentication confirmed.

---

## 🧠 Key Learnings

- Declarative pipelines help simplify CI/CD logic and make workflows reproducible across environments.
- Agent labels ensure that builds run on the correct infrastructure, improving resource isolation and control.
- Credential management is essential for security. Secrets should be stored in Jenkins credentials, not hardcoded.
- Minimal base images like `alpine` reduce image size, improve security posture, and speed up build times.
- Digest-based tagging using `sha256` provides immutability and traceability for pushed Docker images.

---

## 🛡️ Best Practices

- Use multi-stage Docker builds to separate build and runtime layers for cleaner, smaller images.
- Pin base images to specific versions or digests to avoid unexpected changes in future builds.
- Enable Docker layer caching to accelerate rebuilds during iterative development.
- Configure webhook triggers or polling to automate builds on Git commits.
- Validate Dockerfile syntax and lint before pushing to the registry.
- Monitor image sizes and registry pushes to prevent bloated containers and storage issues.

---

## 🧩 Troubleshooting

- If `docker push` fails, verify registry login status and ensure the registry is reachable from the agent.
- If Git checkout fails, confirm credentials, repository URL, and network access from the agent to Gitea.

---

## 🎯 Final Thoughts

This pipeline provides a solid foundation for containerized CI/CD workflows. By combining Git, Docker, and Jenkins with secure practices, you create a reproducible and scalable system. Continue to iterate by adding testing, deployment automation, and observability to evolve it into a production-grade pipeline.

---
