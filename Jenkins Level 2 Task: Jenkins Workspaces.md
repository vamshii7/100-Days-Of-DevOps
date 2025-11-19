# 🚀 Jenkins Level 2 Task: Jenkins Branch-Based Deployment 

Welcome to my DevOps journey! This guide walks step-by-step through configuring a Jenkins job that deploys specific Git branches to a shared storage server. Everything is explained clearly with commands and reasoning so you can follow along.  

<br>  

<img width="749" height="1013" alt="Screenshot 2025-11-18 165253" src="https://github.com/user-attachments/assets/7c66c9e2-3d17-485c-a0cf-984b452bfb46" />  


---

## 🧩 What you’ll achieve
- Create a Jenkins job (`app-job`) that deploys one of several branches (`version1`, `version2`, `version3`) on demand.  
- Use a **choice parameter** to pick the branch during build.  
- Use a **custom workspace per branch** so builds don’t interfere.  
- Clone the repo manually in the shell build step and **scp** the files to the storage server `/var/www/html` where app servers mount the document root.

---

## 🔐 Prerequisites (Checklist)
Before you start, make sure these are in place:

- [x] Jenkins is installed and accessible (Jenkins master control panel).  
- [x] A Jenkins credential exists for `natasha`'s SSH private key (used to connect to `ststor01`). Add it under *Credentials → System → Global credentials*.  
- [x] `natasha` user on `ststor01` has passwordless sudo if you need privileged steps (optional).  
- [x] The storage server `ststor01` has `/var/www/html` and app servers mount it as the document root.  
- [x] The Git repository is reachable from Jenkins: `http://git.stratos.xfusioncorp.com/sarah/web_app.git`  
- [x] The Jenkins master has `git`, `scp`, and `ssh` available on the agent that will run the job (or install them).

---

## 🔧 Job Overview: `app-job`
- **Job type:** Freestyle (or Pipeline if you prefer scripted steps)  
- **Job name:** `app-job`  
- **Parameter:** Choice parameter named `Branch` with values:  
  - `version1`
  - `version2`
  - `version3`  
- **Custom workspace:** `/var/lib/jenkins/${Branch}`  
  - This path ensures each branch has its own workspace. Jenkins will create and use it automatically.

---

## 🧭 Step-by-step: Create the Job  

### 1. Create a new job
1. Open Jenkins → Click **New Item**.  
2. Enter **app-job** as the name.  
3. Select **Freestyle project** (or *Pipeline* if you want to use the Jenkinsfile approach).  
4. Click **OK**.

### 2. Add the Choice Parameter
1. Check **This build is parameterized**.  
2. Click **Add Parameter → Choice Parameter**.  
3. Configure:
   - **Name:** `Branch`  
   - **Choices:** (one per line)  
     ```
     version1
     version2
     version3
     ```
   - **Description:** `Pick the branch to deploy (version1/version2/version3)`

### 3. Set Custom Workspace
1. Under **Advanced Project Options**, find **Use custom workspace**.  
2. Set **Directory** to:
   ```
   /var/lib/jenkins/${Branch}
   ```
3. This creates a clean workspace per branch and prevents overlap.

### 4. Disable Git SCM
- Since we clone manually in the shell step, you can leave the *Source Code Management* section blank (or disable it). This gives full control to your shell script.

---

## 🛠️ Build Step: Shell Script  

Add a **Build → Execute shell** step and paste the following script:

```bash
#!/bin/bash
echo "Deploying branch: ${Branch}"

rm -rf ./*

# Clean workspace
# Deletes everything in the Jenkins workspace.
# Important because each build should start fresh.
# ⚠️ Safe inside Jenkins workspace, dangerous elsewhere. Never use / or absolute paths here.

# Clone selected branch
git clone -b "${Branch}" http://git.stratos.xfusioncorp.com/sarah/web_app.git .

# Transfer files to storage server
echo "Transferring files to ststor01:/var/www/html"

scp -o StrictHostKeyChecking=no -r ./* natasha@ststor01:/var/www/html/

echo "Deployment complete!"

```

---

## ✅ Verify the Deployment

1. In Jenkins, click **Build with Parameters** → select `Branch = version1` → Build.  
2. Open the **Console Output** to watch the steps and confirm `rsync` succeeded.  
3. On the storage server (ststor01), check `/var/www/html` files and timestamps:
   
   ```bash
   ssh natasha@ststor01 'ls -la /var/www/html | head -n 20'
   ```  
5. Access the app to confirm content (example shows “This is app version 1”).

### 🧪 Web App Validation   

- After deploying `version1`, the app displays:  

  ```This is app version 1```  

  <img width="621" height="120" alt="Screenshot 2025-11-18 174633" src="https://github.com/user-attachments/assets/697b7ec6-7ff8-4795-aed0-d51f4ad8f1e1" /> 
  
---

## 🔍 Troubleshooting (Common issues & fixes)

- **Permission denied when writing to `/var/www/html`**  
  - Ensure `natasha` has write permission. Use:
    ```bash
    sudo chown -R natasha:nginx /var/www/html
    sudo chmod -R 755 /var/www/html
    ```
  - Alternatively, `sudo` rsync on the remote side by pushing to a temp directory and using remote `sudo` to move files (more secure methods exist).

- **Git `dubious ownership` or permission errors**  
  - If Jenkins agent user differs from repo owner, run:
    ```bash
    git config --global --add safe.directory /var/lib/jenkins/${Branch}
    ```
    as the Jenkins user or include it in the script.

- **SSH host key checking failed**  
  - When automating, you might disable strict checking (`-o StrictHostKeyChecking=no`) but better approach is to add Jenkins master's public key to `~/.ssh/known_hosts` of `ststor01`.  

---

## 🔐 Security Best Practices

- Use **Jenkins Credentials** store to handle SSH keys - do not hardcode keys into scripts.  
- Prefer `ssh-agent` or the Jenkins **SSH Credentials** binding for secure key usage.  
- Avoid `StrictHostKeyChecking=no` in production; instead add known hosts securely or manage via configuration management.  
- Run Jenkins agents as a dedicated, non-root user (`jenkins`). Don’t grant more privileges than necessary.  
- Use `rm -rf` with caution; it will remove files on the system. ⚠️ Safe inside Jenkins workspace, dangerous elsewhere. Never use / or absolute paths here.

---

## 🧠 Key Learnings

- **Choice parameters** make deployment flexible - one job supports multiple releases.  
- **Custom workspaces** prevent branch builds from clobbering each other.  
- **Manual git clone** in build steps allows finer control (like retry logic, shallow clones, or submodule handling).  
- **SCP/Rsync over SSH** is simple and reliable for file-based deployments when using a shared storage server.  
- **Permissions and Git trust (safe.directory)** are common stumbling blocks - understand who owns the files and which user Jenkins runs as.

---

## 📘 Next steps (suggestions)
- Convert this to a **Declarative Pipeline** (Jenkinsfile) for versioned CI scripts.  
- Integrate tests: run unit/integration tests in the build before deploy.  
- Use an artifact store or registry (for binaries) if you move beyond static files.  
- Implement rollback strategy: keep previous releases or tagged snapshots to revert quickly.

---

## 🏁 Final checklist before you finish
- [ ] SSH key added to Jenkins and tested.  
- [ ] `/var/lib/jenkins/${Branch}` workspaces are clean and writable.  
- [ ] `scp` connectivity to `ststor01` tested manually.  
- [ ] Jenkins job builds successfully and console output shows `Deployment complete`.

---
