# 🚀 Day 66: Deploying MySQL on Kubernetes  

Welcome to **Day 66** of the **#100DaysOfDevOps** challenge!  

<br>

Today’s mission was to deploy **MySQL** inside a **Kubernetes cluster** — transitioning from stateless workloads (like Redis from Day 65) to **stateful applications** that require data persistence.  

<br>  

<img width="985" height="1141" alt="Screenshot 2025-11-01 114539" src="https://github.com/user-attachments/assets/46351071-f784-4d04-a9f1-053b5f54e255" />  


---

## 📘 Table of Contents  
1. [Introduction](#-introduction)  
2. [Key Kubernetes Concepts Used](#-key-kubernetes-concepts-used)  
3. [Step-by-Step Implementation](#-step-by-step-implementation)  
4. [Verification](#-verification)  
5. [Best Practices](#-best-practices)  
6. [Key Learnings](#-key-learnings)  

---

## 💡 Introduction  

Deploying **MySQL on Kubernetes** brings a new layer of complexity because databases require **persistent storage** and **secure credentials**.  
In this task, we explored how to:  
- Create persistent volumes for MySQL data  
- Secure credentials using Kubernetes Secrets  
- Deploy MySQL in a declarative, version-controlled way  
- Expose the database using a NodePort Service  

---

## 🧩 Key Kubernetes Concepts Used  

| Concept | Purpose |
|----------|----------|
| **PersistentVolume & Claim** | Ensures data persists beyond pod restarts |
| **Secret** | Stores sensitive data like database passwords |
| **Deployment** | Manages the lifecycle of MySQL pods |
| **Service (ClusterIP)** | Enables stable networking between application components |
| **Environment Variables** | Pass configuration dynamically to the container |

---

## ⚙️ Step-by-Step Implementation  

### 1️⃣ Create Secrets for MySQL Credentials  

<br>

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-root-pass
stringData:
  password: YUIidhb667
---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-user-pass
stringData:
  username: kodekloud_cap
  password: B4zNgHA7Ya
---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-db-url
stringData:
  database: kodekloud_db5
  ```
<br>

### 2️⃣ Create a PersistentVolume & PersistentVolumeClaim  

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 250Mi
  storageClassName: "standard" 
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data/mysql
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: "standard" 
  resources:
    requests:
      storage: 250Mi
  volumeName: mysql-pv

```

### 3️⃣ Deploy MySQL  

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: MySQL
  template:
    metadata:
      labels:
        app: MySQL
    spec:
      containers:
        - name: mysql-container
          image: mysql:8.0
          volumeMounts:
            - name: persistentvolume
              mountPath: /var/lib/mysql
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-root-pass
                  key: password
            - name: MYSQL_DATABASE
              valueFrom:
                secretKeyRef:
                  name: mysql-db-url
                  key: database
            - name: MYSQL_USER
              valueFrom:
                secretKeyRef:
                  name: mysql-user-pass
                  key: username
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-user-pass
                  key: password
      volumes:
        - name: persistentvolume
          persistentVolumeClaim:
            claimName: mysql-pv-claim  
```
<br>

### 4️⃣ Expose MySQL with a NodePort Service  

<br>

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  type: NodePort
  ports:
    - port: 3306
      targetPort: 3306
      nodePort: 30007
  selector:
    app: MySQL  
```
<br>

---  
## Apply all the manifests

```
kubectl apply -f mysql-pv-pvc.yaml  
kubectl apply -f mysql-secrets.yaml  
kubectl apply -f mysql-deploy.yaml  
kubectl apply -f mysql-service.yaml  
```
<br>

<img width="411" height="164" alt="Screenshot 2025-11-01 115310" src="https://github.com/user-attachments/assets/b9e925dc-ab8a-48cb-b0e6-d5aacaa32c8a" />  <br>

---  

## 🔍 Verification  

```
kubectl get all 
```
<br>

<img width="1096" height="265" alt="Screenshot 2025-11-01 115409" src="https://github.com/user-attachments/assets/f7dafe52-9a89-492d-b724-32223fb04087" />  

<br>  
<br>

```
kubectl describe deploy/httpd-deploy  
```
<br>

<img width="815" height="803" alt="Screenshot 2025-11-01 115613" src="https://github.com/user-attachments/assets/e05ff5ed-0a9b-4b26-8409-a02c5b6c9c7b" />  

---

## 🧠 Best Practices  

- Use Secrets for passwords — Never hardcode credentials in YAML.  
- Attach PVCs to store data — Prevents data loss during restarts.  
- Use ConfigMaps for non-sensitive configs (like MySQL tuning parameters).  
- Limit CPU/Memory resources to maintain cluster efficiency.  
- Enable readiness probes for production setups.

---

## 🎓 Key Learnings  

- Stateful workloads require persistent storage and careful orchestration.  
- Declarative configurations make DB deployments repeatable and reliable.  
- Managing Secrets and PVCs are critical DevOps skills for Kubernetes.  

---

## 💭 Final Thoughts

Each passing day in this challenge reminds me that DevOps mastery is built one YAML, one container, and one deployment at a time.
The #100DaysOfDevOps journey isn’t just about completing tasks — it’s about discipline, curiosity, and community.

Deploying MySQL on Kubernetes wasn’t just another configuration exercise — it was a deep dive into how real-world systems store and safeguard data, even in a dynamic cloud-native world.

As I continue this journey, I’m realizing that DevOps is a mindset — a blend of engineering excellence, automation, and collaboration.
Every day adds a new layer of understanding, and every failure teaches something that books never will.

So here’s to persistence — both in storage and in learning.
💪 Keep deploying. Keep breaking. Keep improving.

---  

**Task Completed Successfully 🎉**  

<img width="1849" height="947" alt="Screenshot 2025-11-01 115704" src="https://github.com/user-attachments/assets/d0c598b7-3982-44d6-b4b1-00d2308d6bb5" />

---
