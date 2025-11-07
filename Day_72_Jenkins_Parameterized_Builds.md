# 🚀 Day 72 - Jenkins Parameterized Builds  

> 🎯 **Objective:** Learn how to configure and run **parameterized builds** in Jenkins - enabling users to pass input values dynamically (like stage names or environments) during build time.

---

## 🧩 Task Overview  
In this session, you’ll:  
✅ Create a **new Jenkins job**.  
✅ Define **build parameters** (String and Choice).  
✅ Use those parameters inside your build steps or scripts.  
✅ Trigger the build via **“Build with Parameters”** and verify that Jenkins uses the provided values.

---

## ⚙️ Preconditions  
Before starting, make sure:  
- 🖥️ **Jenkins Server** is up and running, and you can access the Jenkins UI.  
- 🔐 You have privileges to **create and configure** jobs.  
- 📦 **No extra plugins** are required — parameterized builds are supported out-of-the-box.  
  > 💡 Tip: From Jenkins docs — “Enable the option *This build is parameterized* in your job settings.”  
- 🧠 Basic familiarity with executing shell/batch commands in Jenkins.

---

## 🧠 Step-by-Step Instructions  

### 🏗️ Step 1: Create a New Jenkins Job  
1. From the Jenkins dashboard, click **“New Item”**.  
2. Enter a job name - for example: **`parameterized-job`**.  
3. Choose **Freestyle project** and click **OK**.
<br>

<img width="893" height="303" alt="image" src="https://github.com/user-attachments/assets/d0d66797-57ab-4f2f-8f4e-544dd935df1f" />  

---

### ⚙️ Step 2: Add Build Parameters  
1. On the job configuration page, under **General**, check **“This project is parameterized.”**  
2. Click **Add Parameter → String Parameter**  
   - **Name:** `Stage`  
   - **Default Value:** `Build`  
   - **Description:** _Which stage to run (e.g., Build, Test, Deploy)_

<img width="977" height="477" alt="image" src="https://github.com/user-attachments/assets/9adbcb8f-09a0-4580-81a0-85ab9faff9a1" />  
<img width="728" height="353" alt="image" src="https://github.com/user-attachments/assets/a4d88366-768a-4a38-986d-087338625ff0" />


3. Click **Add Parameter → Choice Parameter**  
   - **Name:** `env`  
   - **Choices (one per line):**  
     ```
     Development  
     Staging  
     Production
     ```  
   - **Description:** _Select the deployment environment_  
<img width="630" height="313" alt="image" src="https://github.com/user-attachments/assets/89f9a40a-e2fe-4f2c-86ff-6d94bd84dc19" />  


> 💡 You can also add other types like **Boolean**, **Multi-line String**, or **Credentials**.

---

### 🧱 Step 3: Configure the Build Step  
1. Scroll to the **Build** section.  
2. Click **Add build step → Execute shell**.  
3. Add the following commands:  

   ```bash
   echo "Stage parameter value: $Stage"
   echo "Environment selected: $env"
   ```

4. Click **Save** to store the configuration.  

<img width="742" height="363" alt="image" src="https://github.com/user-attachments/assets/25a699d9-9154-4db6-8267-aac0714b30dc" />  

---

### ▶️ Step 4: Trigger the Parameterized Build  
1. On the job’s main page, click **Build with Parameters**.  
2. Enter values, for example:  
   - `Stage = Build`  
   - `env = Staging`  
3. Click **Build** 🚀

<img width="755" height="392" alt="image" src="https://github.com/user-attachments/assets/da704a6b-e419-4f94-b824-f04d2238309b" />  

---

### 📜 Step 5: Verify the Build Output  
1. In the **Build History**, click on the latest build (e.g., `#1`).  
2. Open **Console Output** to see results:  

   ```
   Stage parameter value: Build  
   Environment selected: Staging
   ```
<img width="1056" height="383" alt="image" src="https://github.com/user-attachments/assets/34def9f5-c093-4b45-ba92-476e04c56ba7" />  

✅ Jenkins successfully used your input parameters during the build!  

---

## 💡 Discussion & Best Practices  

| 🧠 Tip | Description |
|--------|--------------|
| 🎯 **Flexibility** | Parameterizing builds allows a single job to handle multiple environments or stages. |
| 🧾 **Clarity** | Use descriptive names and sensible defaults for better usability. |
| 🎛️ **Controlled Inputs** | Use **Choice Parameters** to prevent invalid values. |
| 🔐 **Secure Inputs** | For passwords or tokens, use **Credentials Parameters** and mask them. |
| 🌐 **Remote Triggers** | Trigger builds with parameters using the Jenkins API:  <br> `http://<JENKINS_URL>/job/<JOB_NAME>/buildWithParameters?PARAM=value` |
| ⚙️ **Environment Variables** | All parameters are available as environment variables inside the job. |
| 🧩 **Scalability** | For complex CI/CD workflows, use shared parameter templates or Pipeline syntax. |

---

## 🏁 Wrap-Up  

- 🏗️ Created a **parameterized Jenkins job**.  
- ✏️ Defined and used **String** & **Choice** parameters.  
- ▶️ Triggered builds with **custom values**.  
- 🔍 Verified that parameters were passed correctly.  
- 🚀 Understood how parameters improve CI/CD **reusability** and **automation**.

---

## 💬 Reflection Questions  
> 💭 Think about these scenarios to deepen your understanding:

1. What happens if you **don’t provide a value** for a parameter (uses default)?  
2. How could you use parameters to **choose Git branches** dynamically (e.g., `git_branch`)?  
3. How can sensitive data (like credentials or tokens) be handled securely in parameters?  

---

✨ **Next Challenge:**  
Try extending your job with:  
- 🧰 Boolean or Credentials parameters.  
- 🧪 A pipeline that uses `params.<ParameterName>` in a Jenkinsfile.  
- 🔄 A parameter-driven deploy script to simulate real-world DevOps workflows.

---
