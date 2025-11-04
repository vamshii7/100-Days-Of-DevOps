# 🧑‍💻 Day 63: Deploying Iron Gallery App on Kubernetes <br>  

Successfully completed another hands-on task as part of the **100 Days of DevOps Challenge** — this time focused on deploying a multi-container application (Iron Gallery) on Kubernetes! 🚀    


<img width="1895" height="1001" alt="Screenshot 2025-10-29 125703" src="https://github.com/user-attachments/assets/c4542dce-042c-46af-ad2e-e77e09c43365" />    <br>

---  

## 🧩 Objective    

<img width="722" height="1221" alt="image" src="https://github.com/user-attachments/assets/4c3fc69e-a198-4ad7-8846-28710b15ca78" />    <br>

The task was to deploy a photo gallery web application (`Iron Gallery`) along with its database (`MariaDB`) on a Kubernetes cluster.    

This involved creating:  
- A **namespace**
- Two **Deployments** (for frontend and database)
- Two **Services** (ClusterIP for DB and NodePort for frontend)
- Proper **volume mounts** and **environment configurations**  <br>

---   

## ⚙️ Steps to Deploy  

### 1. Create a Namespace  
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: iron-namespace-xfusion
```

### 2. Create the Frontend Deployment  
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iron-gallery-deployment-xfusion
  namespace: iron-namespace-xfusion
  labels:
    run: iron-gallery
spec:
  replicas: 1
  selector:
    matchLabels:
      run: iron-gallery
  template:
    metadata:
      labels:
        run: iron-gallery
    spec:
      containers:
      - image: kodekloud/irongallery:2.0
        name: iron-gallery-container-xfusion
        resources:
          limits:
            cpu: "50m"
            memory: "100Mi"
        volumeMounts:
        - name: config
          mountPath: /usr/share/nginx/html/data
        - name: images
          mountPath: /usr/share/nginx/html/uploads
      volumes:
      - name: config
        emptyDir: {}
      - name: images
        emptyDir: {}
```

### 3. Create the Database Deployment  
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iron-db-deployment-xfusion
  namespace: iron-namespace-xfusion
  labels:
    db: mariadb
spec:
  replicas: 1
  selector:
    matchLabels:
      db: mariadb
  template:
    metadata:
      labels:
        db: mariadb
    spec:
      containers:
      - name: iron-db-container-xfusion
        image: kodekloud/irondb:2.0
        env:
        - name: MYSQL_DATABASE
          value: "database_blog"
        - name: MYSQL_ROOT_PASSWORD
          value: "R00tP@ssw0rd!2025"
        - name: MYSQL_USER
          value: "appuser"
        - name: MYSQL_PASSWORD
          value: "C0mpl3xP@ss!2025"
        volumeMounts:
        - name: db
          mountPath: /var/lib/mysql
      volumes:
      - name: db
        emptyDir: {}
```

### 4. Create Services for Communication  
```yaml
apiVersion: v1
kind: Service
metadata:
  name: iron-db-service-xfusion
  namespace: iron-namespace-xfusion
spec:
  selector:
    db: mariadb
  ports:
  - targetPort: 3306
    port: 3306
    protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: iron-gallery-service-xfusion
  namespace: iron-namespace-xfusion
spec:
  type: NodePort
  selector:
    run: iron-gallery
  ports:
  - targetPort: 80
    port: 80
    nodePort: 32678
    protocol: TCP
```

---  

## 🔍 Verification  

### Once the manifests were applied using:  
```bash
kubectl apply -f deploy.yaml
# similarly apply all manifests
```

### Check resources:  
```bash
kubectl get all -n iron-namespace-xfusion
```   

### ✅ All pods and services were running successfully!  <br>  
<img width="2559" height="1356" alt="Screenshot 2025-10-29 124801" src="https://github.com/user-attachments/assets/9a10920a-70a2-4dfe-9cb9-f0742933d176" />

---

## 🌐 Accessing the Application  

Since the frontend service was exposed as a **NodePort (32678)**, it was accessible via:  
```
http://<NodeIP>:32678
```
or directly through the provided lab URL.  <br>  

## 🔐 Login Details  

Use The Credentials you have created to login (Optional)  

| Field           | Value                      |
|-----------------|----------------------------|
| Database Host   | iron-db-service-xfusion |
| Database Username | appuser                   |
| Database Password | C0mpl3xPgss!2025          |
| Database Name   | database_blog               |
| Table prefix    | (leave blank or set if needed) |  <br>  

<img width="2559" height="1356" alt="Screenshot 2025-10-29 124827" src="https://github.com/user-attachments/assets/7ecb3b06-a491-47a3-842b-19bdf71d64c9" />  <br>  

### Create Username and Password for your installation  

| Field     | Value    |
|-----------|----------|
| Username  | appuser  |
| Password  | C0mpl3xPgss!2025      |  <br>  

<img width="2550" height="1349" alt="Screenshot 2025-10-29 114000" src="https://github.com/user-attachments/assets/c179282d-3dd4-4739-9ce1-42ec86748aaa" />  <br>  

### Once created you should see below page  <br>

<img width="2550" height="1349" alt="Screenshot 2025-10-29 114038" src="https://github.com/user-attachments/assets/023d8332-94ec-469c-a378-fef0565fb2b6" />  

---

## 📚 Key Learnings  

- Configured **multi-container applications** on Kubernetes.
- Understood **inter-service communication** using ClusterIP and NodePort.
- Managed **volumes (emptyDir)** for ephemeral storage.
- Deployed a **stateful component (MariaDB)** in a lightweight setup.

---

## 🏁 Conclusion  

This challenge reinforced the power of Kubernetes in orchestrating microservices-based applications.  
Bringing together networking, storage, and environment management — all declaratively — truly showcases DevOps automation at its best. 💪

---

### 📸 Screenshots Recap  
| Screenshot | Description |
|-------------|-------------|
| <img width="2559" height="1356" alt="Screenshot 2025-10-29 124801" src="https://github.com/user-attachments/assets/de8ff969-8de7-45a4-a634-881f00bccd23" /> | Terminal and Task UI |
| <img width="2557" height="1315" alt="Screenshot 2025-10-29 125650" src="https://github.com/user-attachments/assets/85e0af53-d4ac-4702-9936-ef0c7c79265d" /> | Task Completed Successfully |

---
 
