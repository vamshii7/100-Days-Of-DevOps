# 🚀 Day 76: Jenkins Project Security

> *"Access control in CI/CD isn’t just about restriction - it’s about enabling collaboration with the right boundaries."*  

## 🧩 **Task Overview** 

<br>

<img width="752" height="1083" alt="Screenshot 2025-11-11 141254" src="https://github.com/user-attachments/assets/6ebf5ef4-2140-4792-af97-2bafbdbcc6fe" />  

<br>  

Today’s challenge builds on Jenkins security by granting **project‑specific permissions** to new developers. We’ll configure the existing `Packages` job so that two users (`sam` and `rohan`) have tailored access aligned with their responsibilities.  

---

## 📋 Prerequisites
- A running Jenkins server
- Admin login credentials (`admin / Adm!n321`)
- Installed **Matrix Authorization Strategy Plugin**

---

## 🔑 Step 1: Install Matrix Authorization Plugin
1. Log in as `admin`.
2. Navigate to **Manage Jenkins → Manage Plugins**.
3. Search for **Matrix Authorization Strategy Plugin**.
4. Install and restart Jenkins.
   
   <img width="1014" height="350" alt="Screenshot 2025-11-11 141356" src="https://github.com/user-attachments/assets/e2933d71-7b57-4e90-b4f7-06c170325087" />  


---

## ⚙️ Step 2: Enable Project-based Authorization
1. Go to **Manage Jenkins → Security**.
2. Under **Authorization**, select **Project-based Matrix Authorization Strategy**.
3. Save changes.
   
   <img width="1095" height="406" alt="Screenshot 2025-11-11 141851" src="https://github.com/user-attachments/assets/9e3b3c61-5c8a-49a4-8aad-dcdf33b38e0a" />


---

## 🛠 Step 3: Configure Job Permissions
1. Open the existing job **Packages**.
2. Click **Configure → Enable project-based security**.
3. Select **Inherit permissions from parent ACL**.
4. Add users and assign permissions:

   - **sam**:  
     - Build  
     - Configure  
     - Read  

   - **rohan**:  
     - Build  
     - Cancel  
     - Configure  
     - Read  
     - Update  
     - Tag  

5. Save the configuration.  
   
   <img width="809" height="480" alt="Screenshot 2025-11-11 142135" src="https://github.com/user-attachments/assets/21175cf1-f6a6-4bd5-97a5-11adc6fafe4b" />


---

## ✅ Summary

  | User   | Permissions Granted |
  | ------ | ------------------- |
  | **sam**   | Build, Configure, Read |
  | **rohan** | Build, Cancel, Configure, Read, Update, Tag |  

This setup ensures both developers have the **least privilege required** to contribute effectively without risking unintended changes.

---

## 💡 Best Practices & Key Learnings
- **Least Privilege Principle**: Assign only the permissions needed.  
- **Matrix Authorization Strategy**: Provides fine‑grained control at job level.  
- **Documentation & Screenshots**: Always capture changes for audit and review.  
- **Test Access**: Verify by logging in as each user to confirm permissions.  

---

## 🎯 Final Thoughts
- By configuring **project‑based security** in Jenkins, we enable safe collaboration while protecting critical CI/CD pipelines.  
- This exercise reinforces the importance of **role‑based access control** in DevOps workflows.

> *"A secure pipeline builds confidence and confident teams build better software."* 💪
