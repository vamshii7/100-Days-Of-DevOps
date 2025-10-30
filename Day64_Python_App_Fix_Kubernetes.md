# 🐍 Day 64 of #100DaysOfDevOps  

### 🎯 **Focus:** Fixing a Misconfigured Python (Flask) App Deployment on Kubernetes  

---

## 🧩 Problem Statement  <br>

<img width="1001" height="538" alt="Screenshot 2025-10-30 103653" src="https://github.com/user-attachments/assets/a020fc1c-f261-479e-81bb-65816c3634e5" />  <br>

## 🧠 Root Cause  

After investigating the deployment using:  
```bash  
kubectl get all -n default
```
The pod showed:  
```
STATUS: ImagePullBackOff
```  
<p align="center">  
<img width="650" height="215" alt="image" src="https://github.com/user-attachments/assets/4aaa31d9-8536-4291-befe-1d7476f006d0"  />  <br>
  
This error indicates that the **container image could not be pulled** — likely due to a typo or incorrect image name.  <br>

Use below commands to cehck the pod logs, existing image name and container port exposed.  <br>
```
kubectl describe pod/<pod-name>
kubectl describe deployment/<deployment-name>
kubectl describe service/<service-name>
```  

Additionally, the **Service configuration** did not match the container's internal port, which prevented external access.  

### Your task:    
- Fix the image and ports configuration.  
- Ensure the application is accessible on the specified **NodePort (32345)**.  
    
---

## 🛠️ Steps to Fix  

### **1️⃣ Correct the Image Name**  

The existing image reference in the Deployment was incorrect, update it to the valid Flask demo image as specified in Task.  

Before:  
```yaml
image: poroko/flask-app-demo
```
After:  
```yaml
image: poroko/flask-demo-app
```

Edit the deployment directly:  
```bash
kubectl edit deployment python-deployment-xfusion
```

---

### **2️⃣ Update the Service Ports**  

The Flask app runs on **port 5000** inside the container. The **NodePort** should be exposed as **32345** for external access.  

Edit the service:  
```bash
kubectl edit service python-service-xfusion
```

Correct configuration:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: python-service-xfusion
spec:
  type: NodePort
  selector:
    app: python-app
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
      nodePort: 32345
```

---

### **3️⃣ Verify Deployment**  

Once changes were applied:  
```bash
kubectl get all -n default  
```  

<img width="648" height="317" alt="image" src="https://github.com/user-attachments/assets/73ff4247-de56-4c41-943e-c24fa5b574f0" />  <br>

---

### **4️⃣ Test Application Access**  

Access the application using the KodeKloud Lab URL:  

✅ Output:  


<img width="811" height="184" alt="Screenshot 2025-10-30 103806" src="https://github.com/user-attachments/assets/4c17404e-411a-486a-ba81-f0681e7204d6" />  <br>


---

## 📚 Key Learnings  

| Concept | Description |
|----------|-------------|
| **ImagePullBackOff** | Occurs when Kubernetes cannot pull an image from the registry (typo, auth, or tag issues). |
| **NodePort Service** | Exposes a service on each node’s IP at a static port, allowing external access to the cluster. |
| **TargetPort** | The port on the container to which the service forwards traffic. |
| **kubectl edit** | A quick way to modify existing Kubernetes resources live without redeploying YAML files. |  

---

## 🚀 Key Takeaways  

- Always double-check **image names** before deploying — even small typos cause pull failures.  
- Verify port mappings between **container → service → nodePort**.  
- Use `kubectl describe` for detailed error messages when debugging pod issues.  
- `kubectl edit` is powerful for real-time debugging in live clusters.  
- Always confirm access via browser or `curl` to ensure connectivity.  

---

## 🧭 Best Practices  

- Use **ConfigMaps** or environment variables for dynamic app configuration.  
- Pin image versions (e.g., `poroko/flask-demo-app:v1`) to avoid unexpected updates.  
- Prefer **Helm or Kustomize** for managing reusable Kubernetes manifests.  
- Integrate **readiness & liveness probes** to monitor container health.  
- Implement **RBAC and namespaces** for better isolation and control.  

---

## 🏁 Final Output  

**Task Completed Successfully 🎉**  

<img width="1862" height="918" alt="Screenshot 2025-10-30 103828" src="https://github.com/user-attachments/assets/8d00cb3c-453d-4890-8a4a-e9d53e9fab72" />  <br>


> ✔ Fixed Deployment Image  
> ✔ Corrected Service NodePort  
> ✔ Verified Flask App Response  
> ✔ Application accessible on:  
> `https://32345-port-goxg5ac23mglgufb.labs.kodekloud.com`

---

## 🧑‍💻 Author  

**Vamshi Krishna**  
*100 Days of DevOps - Day 64*  
📅 *Fix Python App Deployment on Kubernetes*  
