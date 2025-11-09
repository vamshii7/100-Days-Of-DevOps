# 🚀 Kubernetes Level-3 Task: Deploy Iron Gallery App on Kubernetes

> **Objective:**  
> Deploy a complete two-tier Iron Gallery Application on Kubernetes - consisting of a web frontend (Nginx) and a MariaDB backend - with proper namespaces, resources, and service exposure.

<img width="754" height="1180" alt="Screenshot 2025-11-08 192925" src="https://github.com/user-attachments/assets/3d7cb06a-3382-4a3d-a015-8bd8ed688153" />

---

## 🧩 Kubernetes Architecture Overview  

<img width="2183" height="617" alt="image" src="https://github.com/user-attachments/assets/12b5f77a-fa4e-49d7-be30-33f07bd5bd8c" />  

## 📦 Step-by-Step Deployment  
<details open> <summary><b>1️⃣ Create a Namespace</b></summary>

This namespace logically groups the Iron Gallery application and its database components.
<br>

```iron-namespace-devops.yaml```
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: iron-namespace-devops
```
</details>  

<details open> <summary><b>2️⃣ Deploy Iron Gallery (Frontend App)</b></summary>  

This deployment runs the web front-end for the Iron Gallery app using Nginx.  
It mounts two temporary volumes for configuration and image uploads.  
<br>
```iron-gallery-deployment-devops.yaml```  
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iron-gallery-deployment-devops
  namespace: iron-namespace-devops
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
      - name: iron-gallery-container-devops
        image: kodekloud/irongallery:2.0
        resources:
          limits:
            cpu: 50m
            memory: 100Mi
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
</details>  

<details open> <summary><b>3️⃣ Deploy Iron DB (MariaDB Database)</b></summary>  

This deployment runs a MariaDB instance to serve as the backend database for Iron Gallery.
All credentials and environment variables are configured as plain text for demonstration.  
<br>
```iron-db-deployment-devops.yaml```  
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iron-db-deployment-devops
  namespace: iron-namespace-devops
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
      - name: iron-db-container-devops
        image: kodekloud/irondb:2.0
        env:
        - name: MYSQL_DATABASE
          value: database_web
        - name: MYSQL_ROOT_PASSWORD
          value: D@ta3ase
        - name: MYSQL_PASSWORD
          value: D@ta3ase
        - name: MYSQL_USER
          value: dbadmin
        volumeMounts:
        - name: db
          mountPath: /var/lib/mysql
      volumes:
      - name: db
        emptyDir: {}
```
</details>  

<details open> <summary><b>4️⃣ Create Service for Database (ClusterIP)</b></summary>  

This service provides internal connectivity between Iron Gallery and Iron DB.
It exposes port 3306 within the cluster only.  
<br>
```iron-db-service-devops.yaml```  
```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: iron-namespace-devops
  name: iron-db-service-devops
spec:
  selector:
    db: mariadb
  ports:
  - port: 3306
    targetPort: 3306
    protocol: TCP
  type: ClusterIP
```
</details>  

<details open> <summary><b>5️⃣ Create Service for Web App (NodePort)</b></summary>  

This service exposes the Iron Gallery app externally through NodePort 32678.
It allows users to access the frontend via browser or curl.  
<br>
```iron-gallery-service-devops.yaml```  
```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: iron-namespace-devops
  name: iron-gallery-service-devops
spec:
  selector:
    run: iron-gallery
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    nodePort: 32678
    protocol: TCP
```
</details>    

---  

## 🧠 Verification Commands  
```bash
kubectl apply -f <file-name>.yaml
kubectl get all -n iron-namespace-devops
```

<img width="953" height="483" alt="image" src="https://github.com/user-attachments/assets/cd9d1451-c634-4803-9bc8-20e5c7de3cfe" />  
<img width="865" height="313" alt="image" src="https://github.com/user-attachments/assets/822af629-1d75-4e56-bb47-9b1547db7ad9" />  

### ✅ Outcome  

The Iron Gallery App is successfully deployed on Kubernetes with isolated namespaces, functional services, and complete connectivity between frontend and database.  

---

## 🧠 Final Thoughts & Learnings

This task was a great hands-on exercise in deploying a multi-tier application using pure Kubernetes manifests — no Helm, no operators.  

**Here are some of the takeaways:**
- Namespaces help isolate workloads logically, improving manageability in shared clusters.
- EmptyDir volumes are ideal for transient data or test scenarios but not suitable for persistent production storage.
- The NodePort service made the frontend accessible externally, reinforcing how internal (ClusterIP) and external (NodePort) service types complement each other.
- Resource limits (cpu, memory) are crucial even for lab apps — they help Kubernetes manage fair usage and prevent overconsumption.
- Finally, verifying pod-to-pod communication across services strengthens understanding of service discovery in Kubernetes.  

> 💡 This exercise reinforced how Kubernetes abstracts infrastructure concerns while offering immense flexibility for deployment topologies. Each manifest here - namespace, deployment, and service - fits like a Lego piece in the broader puzzle of container orchestration.  
---  
### 🎉 Your Iron Gallery App is now live on Kubernetes! Another step closer to mastering Kubernetes the DevOps way!  
---


