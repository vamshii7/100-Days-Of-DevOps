# 🚀 Day 70: Configure Jenkins User Access  

> _“Security isn’t about restriction — it’s about enabling the right access for the right people.”_  

- Today marks another step forward in our **#100DaysOfDevOps** journey!  
In this challenge, we focused on **Jenkins access control**, ensuring that every user has just the permissions they need — nothing more, nothing less.  

- This hands-on lab demonstrates how to configure **Project-based Matrix Authorization Strategy** in Jenkins.  
By the end, you’ll have a hardened Jenkins setup where the `admin` user retains **full control** and a new user `rose` has **read-only visibility** across the system.

---

<br>

<img width="954" height="871" alt="Screenshot 2025-11-05 140525" src="https://github.com/user-attachments/assets/c96a3691-b2d7-4b8b-8c7e-edacc3b8ec07" />  
<br>

---

## 📌 Prerequisites
- A running Jenkins server  
- Admin login credentials  
- Basic familiarity with Jenkins UI  

---

## 🧩 Step 1: Create a New User
1. Log in as `admin`.  
2. Go to **Manage Jenkins → Users → Create User**.  
3. Fill in the details:  
   - Username: `rose`  
   - Password: `YchZHRcLkL`  
   - Full name: `Rose`  

<img width="816" height="313" alt="image" src="https://github.com/user-attachments/assets/39adea7c-7a28-4bfd-9500-24ce85445108" />  
<br>  

<img width="812" height="644" alt="image" src="https://github.com/user-attachments/assets/4a3fdcbf-90b1-4d92-844f-8f0c5cf28ba4" />  

---

## ⚙️ Step 2: Enable Project-based Matrix Authorization  
1. Navigate to **Manage Jenkins → Security**.  
2. Under **Authorization**, select **Project-based Matrix Authorization Strategy**.  
3. If not visible, install the **Matrix Authorization Strategy Plugin** via **Manage Plugins**.  

<img width="1191" height="342" alt="Screenshot 2025-11-05 145614" src="https://github.com/user-attachments/assets/079ef162-e024-447e-b950-3ff41a7a3c89" />  

<img width="1826" height="636" alt="image" src="https://github.com/user-attachments/assets/0f512fd4-20a4-4a9d-a811-2b970542ebdb" />  

<img width="1156" height="550" alt="image" src="https://github.com/user-attachments/assets/f9824829-b521-4107-8a78-20852a7f1710" />

---

## 🔒 Step 3: Configure Permissions  
1. In the **Matrix-based Security** table:  
   - Add `rose` and `admin`.  
   - Grant only **Overall → Read** to `rose`.  
   - Remove all permissions for **Anonymous** and **Authenticated Users**.  
   - Ensure `admin` retains **Overall → Administer**.  

<img width="1455" height="811" alt="Screenshot 2025-11-05 145445" src="https://github.com/user-attachments/assets/2c6fcc96-d0cb-415a-a0ce-b675a7df2d9d" />  
<br>

---

## 🧱 Step 4: Allow Rose to View Jobs  
To let Rose view jobs globally:  
- Open a Job → **Configure → Enable Project-based Security**.  
- Add `rose` and grant **Job → Read** permission only.  
- Do not grant build, configure, or delete privileges.  

<img width="1595" height="800" alt="Screenshot 2025-11-05 145858" src="https://github.com/user-attachments/assets/7f52ded7-9808-439b-a29e-2e51d7b0da69" />  
<br>

---

## 🧪 Step 5: Test Rose’s Access  
1. Log out and log in as `rose`.  
2. Verify:  
   - Rose can **see jobs** and view basic system info.  
   - Rose **cannot build, configure, or delete** any jobs.  

<img width="1346" height="460" alt="Screenshot 2025-11-05 145952" src="https://github.com/user-attachments/assets/862f0e3c-4b54-4553-95cb-1e34c5378375" />  
<br>

---

## ✅ Summary
| User | Access Level | Permissions |
|------|---------------|-------------|
| **admin** | Full Control | Overall → Administer |
| **rose** | Read-only | Overall → Read, Job → Read |
| **Anonymous / Authenticated Users** | None | — |

This configuration enforces **principle of least privilege**, improving Jenkins security posture and reducing risks from unintended changes.

---

## 💡 Best Practices & Key Learnings

- 🔐 **Least Privilege Principle:** Grant users only the permissions they need.  
- 🧩 **Matrix Authorization Strategy:** Offers fine-grained access control over users and resources.  
- 🔁 **Regular Permission Reviews:** Revisit roles periodically to ensure access stays relevant.  
- 🧠 **Document Everything:** Screenshots or Loom recordings make future audits much easier.  
- ⚡ **Test with Non-Admin Users:** Always verify access from the user’s perspective.

---

## 🎯 Final Thoughts

This challenge emphasized the importance of **role-based access control** in CI/CD environments.  
Proper permission configuration isn’t just a security measure — it’s a foundation for safe collaboration.  

By implementing these steps, you ensure Jenkins remains **secure, auditable, and team-friendly**, setting the stage for scalable DevOps workflows.  

> “A secure pipeline builds confidence — and confident teams build better software.” 💪  

---
