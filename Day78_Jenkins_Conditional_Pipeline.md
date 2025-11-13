# Day 78: Jenkins Conditional Pipeline

In this task, we created a **Jenkins pipeline job** named `devops-webapp-job` to deploy the `web_app` project conditionally based on a branch parameter.  

<img width="827" height="954" alt="Screenshot 2025-11-13 142716" src="https://github.com/user-attachments/assets/7a6c3807-ea1a-48b0-9e0f-c04c757c9cfc" />  


---

## 🔧 Setup Overview

### Jenkins Configuration
1. Open Jenkins and log in with the provided credentials.
2. Add Jenkins slave with the given details [Refer Day 77](https://github.com/vamshii7/100-Days-Of-DevOps/blob/main/Day77_Jenkins_Deploy_Pipeline.md).
3. Create a **Pipeline job** named `devops-webapp-job` (⚠️ not Multibranch Pipeline).
4. Add a **String Parameter** named `BRANCH` with default value `master`.
5. Ensure a Jenkins node is available with the label **ststor01** (Storage Server).

---

### Important configuration notes
- The Jenkins agent process must run as a user that has read/write access to the Git repo under `/var/www/html`.
- Install Git on the agent and ensure `git` is available in PATH.
- Configure `git config --global --add safe.directory /var/www/html` for the `jenkins` user's global git config to avoid "dubious ownership" errors.
- Avoid running the Jenkins agent as `root`.

---

## 🚀 Jenkins Pipeline Script

Below is the complete pipeline script used for the job:

```groovy
pipeline {
    agent { label 'ststor01' }

    parameters {
        string(name: 'BRANCH', defaultValue: 'master', description: 'Branch to deploy (master or feature)')
    }

    stages {
        stage('Deploy') {
            when {
                expression { params.BRANCH == 'master' || params.BRANCH == 'feature' }
            }
            steps {
                script {
                    echo "Starting deployment of branch: ${params.BRANCH}"

                    dir('/var/www/html') {
                        // Ensure git access permissions and safe directory config
                        sh '''
                        # ensure jenkins user trusts the repo (idempotent)
                        git config --global --add safe.directory /var/www/html || true

                        # switch to requested branch and pull latest
                        git checkout ${BRANCH}
                        git pull origin ${BRANCH}
                        '''

                        echo "Successfully deployed branch: ${params.BRANCH} to /var/www/html"
                    }
                }
            }
        }
    }

    post {
        failure {
            echo "Deployment failed for branch: ${params.BRANCH}"
        }
    }
}
```

> Note: The `git config --global --add safe.directory /var/www/html` must be run as the `jenkins` user. If you configured the agent to run under a different user, run this command once for that user (or include equivalent step in a secure provisioning process).

---

## 🔐 Best Practices

- **Run agent as a dedicated non-root user** (`jenkins`) with only necessary privileges.
- **Avoid global `sudo` in pipelines.** If privileged operations are required, handle them via a secured provisioning step or use privileged automation (Ansible) outside the pipeline.
- **Use Jenkins Credentials** for SSH keys and other secrets rather than embedding them in scripts.
- **Validate input parameters.** Consider restricting `BRANCH` using an Active Choice plugin or use an `choice` parameter with allowed values to avoid accidental values.
- **Atomic deployments:** Deploy to a temp directory and swap symlink to minimize partial-site exposure.
- **Zero-downtime reloads:** Use graceful reload/restart for Apache (`apachectl graceful`) if needed, rather than a hard restart.
- **Workspace hygiene:** Clean up temporary files and avoid leaving sensitive artifacts in `/var/www/html`.
- **Use hooks/webhooks:** Trigger the Jenkins job from Gitea/Git webhooks for automation.
- **Backups and rollback:** Keep periodic backups of `/var/www/html` and tag releases so you can roll back quickly.
- **Logging and monitoring:** Capture console logs, and configure alerts on failed builds/deploys.
- **Least privilege for files:** Website files often only need `644` and directories `755`; avoid `777`.

---

## 🧾 Key Learnings from the Task

- Git (v2.35+) enforces safe-directory checks: when Jenkins runs under a different user than the file owner, you may encounter `detected dubious ownership` - fixable by configuring `safe.directory` or aligning ownership.
- Proper filesystem permissions are crucial for CI/CD agents writing to shared document roots. Ownership mismatch often causes permission-denied errors (e.g., `.git/FETCH_HEAD`).
- Running Jenkins agents as non-root reduces blast radius and improves security posture.
- Using a dedicated storage node (ststor01) with the document root mounted to app servers is a simple and effective pattern for static site deployment.
- Pipelines should validate parameters and fail fast on invalid input to avoid unintended deploys.
- Automating agent setup (user creation, SSH keys, git, safe.directory) during provisioning avoids manual steps and reduces configuration drift.

---

## ✅ Validation Steps

1. Ensure `ststor01` agent is connected and labelled `ststor01` in Jenkins.
2. From Jenkins UI → `devops-webapp-job` → Build with Parameters → choose `master` or `feature`.
3. Inspect console output for:
   - Agent connection
   - `git checkout` and `git pull` steps
   - Success message: `Successfully deployed branch: <branch> to /var/www/html`
4. Verify website content via the LB URL (no sub-directory path like `/web_app`).

   <img width="681" height="936" alt="Screenshot 2025-11-13 145006" src="https://github.com/user-attachments/assets/53684ea0-213e-4600-9c09-97bad9553a85" />
   
   <img width="775" height="944" alt="Screenshot 2025-11-13 145036" src="https://github.com/user-attachments/assets/85472973-9ba8-4fcf-9c7f-f3ee5837fb4d" />



---

## 📘 Result

The conditional Jenkins pipeline successfully updates the website content on the Storage Server based on the chosen branch. App Servers immediately reflect these changes as `/var/www/html` is mounted as their document root.  

<img width="1706" height="826" alt="Screenshot 2025-11-13 145404" src="https://github.com/user-attachments/assets/f69e80c9-a09a-446c-b6eb-1085505586d0" />

---
