# Day 80: Jenkins Chained Builds

**Goal:** Configure Jenkins chained builds where an upstream deployment job (`nautilus-app-deployment`) updates the Storage server and - *only if it succeeds* - a downstream job (`manage-services`) restarts Apache (`httpd`) on three app servers.  

<img width="809" height="826" alt="Screenshot 2025-11-17 161102" src="https://github.com/user-attachments/assets/386deadc-1e98-49f4-9810-41afac8019f7" />  

---

## Generate SSH keypair for Jenkins and copy public key to target users

```bash
# create a secure keypair used by Jenkins to SSH to remote servers
ssh-keygen
# private: ~/.ssh/id_rsa
# public:  ~/.ssh/id_rsa.pub
```

**Copy the public key to each remote user's authorized_keys**

Use `ssh-copy-id` (preferred) or manual append if not available.

```bash
ssh-copy-id tony@stapp01
ssh-copy-id steve@stapp02
ssh-copy-id banner@stapp03
ssh-copy-id natasha@ststor01
```

If `ssh-copy-id` is not available:

```bash
# example for natasha
cat ~/.ssh/id_rsa.pub | ssh natasha@ststor01 "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
# repeat for tony, steve, banner
```

**Security note:** Keep the private key safe. We'll upload it into Jenkins Credentials (SSH private key field).

---

## Configure sudoers on each server (minimal and safe)

You must allow each user to restart `httpd` without being prompted for a password.

**On stapp01 (for `tony`):**
```bash
ssh tony@stapp01
sudo visudo
# add the following line:
tony ALL=(ALL) NOPASSWD: /bin/systemctl restart httpd, /bin/systemctl status httpd
```

**On stapp02 (for `steve`):**
```bash
ssh steve@stapp02
sudo visudo
# add:
steve ALL=(ALL) NOPASSWD: /bin/systemctl restart httpd, /bin/systemctl status httpd
```

**On stapp03 (for `banner`):**
```bash
ssh banner@stapp03
sudo visudo
# add:
banner ALL=(ALL) NOPASSWD: /bin/systemctl restart httpd, /bin/systemctl status httpd
```

**On ststor01 (for `natasha`) - allow git/rsync/chown/systemctl if needed for deploy:**
```bash
ssh natasha@ststor01
sudo visudo
# add:
natasha ALL=(ALL) NOPASSWD: /usr/bin/git, /usr/bin/rsync, /bin/chown, /bin/systemctl
```

> Keep entries minimal and specific to reduce risk.

---

## Verify passwordless SSH and sudo

From Jenkins host (or a machine where the private key is present), run:

```bash
ssh -o StrictHostKeyChecking=no tony@stapp01 'whoami; hostname'
ssh -o StrictHostKeyChecking=no steve@stapp02 'whoami; hostname'
ssh -o StrictHostKeyChecking=no banner@stapp03 'whoami; hostname'
ssh -o StrictHostKeyChecking=no natasha@ststor01 'whoami; hostname'
```

Verify sudo capability for allowed commands (should not prompt for a password):

```bash
ssh natasha@ststor01 "sudo git --version || true"
ssh tony@stapp01 "sudo systemctl status httpd"
```

---

## Jenkins: install required plugins

1. Login to Jenkins UI: `http://jenkins:8080` → `admin` / `Adm!n321`
2. Manage Jenkins → Manage Plugins → Available
3. Install:
   - **Credentials Plugin**
   - **SSH Credentials Plugin**
   - **SSH Agent Plugin**
   - **Git Plugin**
4. After installation, click **Restart Jenkins when installation is complete and no jobs are running**.

---

## Add SSH credentials in Jenkins

Add private key as **SSH Username with private key** credentials for each remote user.

1. Manage Jenkins → Manage Credentials → (Global) → Add Credentials
2. Kind: **SSH Username with private key**
   - **Username:** `natasha`
   - **Private Key:** Enter directly - paste the content of `~/.ssh/id_rsa`
   - **ID:** `ssh-natasha` (recommended)

---

## Create upstream job: `nautilus-app-deployment` (Freestyle)

This job will ensure the Storage server `/var/www/html` contains the latest `master` branch from the `sarah/web` repo.

**Steps:**

1. Jenkins → New Item → Enter name: `nautilus-app-deployment` → Freestyle project → OK
2. **Restrict where this project can be run**: optional - leave empty (we're executing commands on remote servers via SSH).
3. **Source Code Management**: *Optional* - because we're doing a remote `git pull` on the storage server, you can leave SCM empty. Alternatively, configure Git here for record-keeping:
   - Git → Repository URL: `https://<GITEA-LBR-URL>/sarah/web`
   - Credentials: use `sarah` credentials if you've added them (`gitea-sarah`)
   - Branch Specifier: `*/master`
4. **Build Triggers**:
   - Recommended: Configure Gitea webhook to notify Jenkins.
   - Fallback: Poll SCM using `H/2 * * * *` (poll every 2 minutes).
5. **Environment**: Enable SSH Agent, choose added ssh credentials.
6. **Build → Execute shell**: Add the exact script below:

```bash
  ssh -o StrictHostKeyChecking=no natasha@ststor01 "cd /var/www/html && git config --global --add safe.directory /var/www/html || true && \
  git fetch origin master && git checkout master && git pull origin master
```

7. **Post-build Actions**:
   - To automatically trigger downstream `manage-services`, add:
     - **Build other projects** → `manage-services`
     - Check **Trigger only if build is stable**

8. Save the job.

   <img width="847" height="1296" alt="Screenshot 2025-11-17 164339" src="https://github.com/user-attachments/assets/fcd78b9a-e429-4b4c-8be0-f05831f47256" />  


---

## Create downstream job: `manage-services` (Freestyle)

This job will restart Apache on the three app servers and should only run after upstream job succeeds.

**Steps:**

1. Jenkins → New Item → Enter name: `manage-services` → Freestyle project → OK
2. **Build Triggers**: Leave blank (it will be triggered by upstream job).
3. **Build → Execute shell**: Add the script below:

```bash
ssh -o StrictHostKeyChecking=no tony@stapp01 "sudo systemctl restart httpd"
ssh -o StrictHostKeyChecking=no steve@stapp02 "sudo systemctl restart httpd"
ssh -o StrictHostKeyChecking=no banner@stapp03 "sudo systemctl restart httpd"
```

4. Save the job.

**Alternative trigger method:** If you didn't use Post-build Actions in the upstream job, you can in `manage-services` configure:  

  - <img width="818" height="1220" alt="Screenshot 2025-11-17 164422" src="https://github.com/user-attachments/assets/3634ce3c-0b68-4077-a628-20463713c723" />  

<br>  

- Build Triggers → **Build after other projects are built** → specify `nautilus-app-deployment` and choose **Trigger only if build is stable**.

Either approach works - Post-build in upstream or "Build after other projects" in downstream.

---

## Test the full flow

1. Run the job `nautilus-app-deployment`.
2. In Jenkins:
   - Open `nautilus-app-deployment` → Console Output → verify steps show git fetch/checkout/pull on `ststor01` and success.  

      <img width="826" height="738" alt="Screenshot 2025-11-17 164357" src="https://github.com/user-attachments/assets/5fecb6fd-2156-47f2-a9dc-8571d50fd0e9" />


   - After upstream finishes successfully, check that `manage-services` is triggered automatically and its console output shows restarts on `stapp01`, `stapp02`, `stapp03`.
  
      <img width="841" height="758" alt="Screenshot 2025-11-17 164447" src="https://github.com/user-attachments/assets/55d2adac-7567-46bf-acac-5f4eddafff87" />  

   
2. Verify the App:
```bash
curl -s https://<LBR-URL> | head -n 20
# or from any app server:
curl -s http://localhost:8080 | head -n 20
```
Expected content: the updated `index.html` message.  

  <img width="638" height="93" alt="Screenshot 2025-11-17 164152" src="https://github.com/user-attachments/assets/02df2fa3-a8ec-4bd8-95cd-d42d999879a5" />  


---

## Troubleshooting & common fixes

**Problem:** `Permission denied (publickey)`  
- Ensure the public key content is in the remote user's `~/.ssh/authorized_keys` and has `600` permissions.  
- Test SSH with `ssh user@host 'echo ok'`.

**Problem:** `sudo: a password is required`  
- Confirm the correct sudoers file exists in `/etc/sudoers.d/` and uses the exact username (e.g., `tony`).  
- Use `sudo visudo` to edit safely.

**Problem:** `git: warning: unsafe repository` or `fatal: detected dubious ownership`  
- Run `git config --global --add safe.directory /var/www/html` on the machine performing git operations (we add it in the upstream script).

**Problem:** Downstream job not triggering  
- Verify upstream post-build action name matches the downstream job name exactly.  
- Check Jenkins logs for errors when trying to trigger downstream.

---

## Checklist (copy & tick)

- [ ] Generated SSH keypair for Jenkins and copied public key to `tony`, `steve`, `banner`, `natasha`
- [ ] Verified SSH connectivity to each host
- [ ] Created sudoers entries on each host for limited commands
- [ ] Installed Jenkins plugins: Credentials, SSH Credentials, SSH Agent, Git
- [ ] Added SSH credentials in Jenkins for `natasha`
- [ ] Created `nautilus-app-deployment` Freestyle job (upstream)
- [ ] Created `manage-services` Freestyle job (downstream)
- [ ] Configured upstream to trigger downstream only on stable/success
- [ ] Performed a Build and verified both jobs run and app shows updated content

---

## Final notes

- Keep sudoers rules minimal to reduce security exposure.  
- Consider using a dedicated `deploy` user across all servers for simpler management in future.  
- For larger environments, consider using configuration management (Ansible) to push sudoers and authorized_keys consistently.

---

