# 🚀 Day 67: Deploy Guestbook Application on Kubernetes

### Welcome to **Day 67** of the **#100DaysOfDevOps** challenge!  

<br>  

<img width="952" height="1226" alt="Screenshot 2025-11-02 123223" src="https://github.com/user-attachments/assets/66a1b5f6-69c3-465b-8d20-ed28161dec4c" />  <br>

<br>  

- Today’s task was to deploy a **multi-tier Guestbook web application** on **Kubernetes** using **Redis** as the backend and a **PHP frontend**.   
- This app allows users to enter messages that are stored and displayed using a Redis database backend.  

---

## 📘 Table of Contents
1. [💡 Introduction](#-introduction)
2. [🖼️ Architecture diagram description](#-architecture-overview)
3. [🏗️ Backend Configuration](#-backend-configuration)
4. [🎨 Frontend Configuration](#-frontend-configuration)
5. [✅ Verification](#-verification)
6. [💡 Key Learnings](#-key-learnings)
7. [🧭 Best Practices](#-best-practices)
8. [🎯 Final Thoughts](#-final-thoughts)

---

## 💡 Introduction

- The **Guestbook app** is a simple web-based system built on the **Redis master-slave replication** model.  
- It demonstrates how **stateful services (Redis)** and **stateless frontends (PHP app)** interact in a Kubernetes environment.
- This is an example that demonstrates how to deploy a PHP frontend backed by a Redis master–slave architecture.  
- This exercise reinforces Deployments, Services, labels/selectors, resource requests, and external access via NodePort. 

---

## 📌 Objective

- Deploy a Redis master (write) and Redis slave (read) backend.  
- Deploy a PHP frontend that interacts with Redis.  
- Expose the frontend externally using a NodePort service.  
- Apply resource requests for predictable scheduling.  

---

## 🏗️ Architecture Overview

```
          +-------------------+
          |  Guestbook UI     |
          |  (Frontend Pods)  |
          +---------+---------+
                    |
                    ↓
          +-------------------+
          |  Redis Slaves     |
          | (Read Operations) |
          +---------+---------+
                    |
                    ↓
          +-------------------+
          |  Redis Master     |
          | (Write Operations)|
          +-------------------+
```

This architecture ensures better performance and separation of read/write operations.

---

# Guestbook Application — Architecture Diagram Description

## 🖼️ Architecture Overview

The Guestbook application is a **multi-tier Kubernetes deployment** consisting of a frontend (PHP) and backend (Redis master–slave). It demonstrates how Kubernetes Services and Deployments work together to provide scalability, redundancy, and stable communication.

---

## 🎨 Frontend (Guestbook)

- **Deployment** with 3 replicas of the PHP client.  
- Exposed externally via **NodePort Service** `frontend` on port **30009**.  
- Users access the application at:  ```http://<NodeIP>:30009```


---

## 🏗️ Backend (Redis)

### Redis Master
- **Deployment** with 1 replica.  
- Exposed internally via **ClusterIP Service** `redis-master` on port **6379**.  
- Handles **all write operations**.

### Redis Slaves
- **Deployment** with 2 replicas.  
- Exposed internally via **ClusterIP Service** `redis-slave` on port **6379**.  
- Handle **read operations** to scale read traffic.

---

## 🔀 Traffic Flow

1. **Client → NodePort Service (frontend)**  
   - External users connect to the frontend service on port 30009.  
   - Requests are load-balanced across the 3 frontend pods.  

2. **Frontend Pods → Redis Master Service**  
   - All **write requests** are routed to the `redis-master` ClusterIP service.  

3. **Frontend Pods → Redis Slave Service**  
   - All **read requests** are routed to the `redis-slave` ClusterIP service.  

---

## 🧩 Key Concept

- **Labels and selectors** tie each Service to the correct set of Pods.  
- This abstracts away dynamic Pod IPs and ensures stable communication between tiers.  
- The architecture demonstrates **scalability (frontend + slaves)** and **consistency (single master for writes)**.

---


## 🔧 Backend Configuration  

### 📄 `guestbook-backend.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-master
  labels:
    app: redis
    role: master
    tier: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
      role: master
      tier: backend
  template:
    metadata:
      labels:
        app: redis
        role: master
        tier: backend
    spec:
      containers:
        - name: master-redis-datacenter
          image: redis
          ports:
            - containerPort: 6379
          resources:
            requests:
              cpu: 100m
              memory: 100Mi
---
apiVersion: v1
kind: Service
metadata:
  name: redis-master
  labels:
    app: redis
    role: master
    tier: backend
spec:
  ports:
    - port: 6379
      targetPort: 6379
  selector:
    app: redis
    role: master
    tier: backend
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-slave
  labels:
    app: redis
    role: slave
    tier: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: redis
      role: slave
      tier: backend
  template:
    metadata:
      labels:
        app: redis
        role: slave
        tier: backend
    spec:
      containers:
        - name: slave-redis-datacenter
          image: gcr.io/google_samples/gb-redisslave:v3
          ports:
            - containerPort: 6379
          env:
            - name: GET_HOSTS_FROM
              value: "dns"
          resources:
            requests:
              cpu: 100m
              memory: 100Mi
---
apiVersion: v1
kind: Service
metadata:
  name: redis-slave
  labels:
    app: redis
    role: slave
    tier: backend
spec:
  ports:
    - port: 6379
      targetPort: 6379
  selector:
    app: redis
    role: slave
    tier: backend
```

---

## 🎨 Frontend Configuration

### 📄 `guestbook-frontend.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: guestbook
      tier: frontend
  template:
    metadata:
      labels:
        app: guestbook
        tier: frontend
    spec:
      containers:
        - name: php-redis-datacenter
          image: gcr.io/google-samples/gb-frontend@sha256:a908df8486ff66f2c4daa0d3d8a2fa09846a1fc8efd65649c0109695c7c5cbff
          ports:
            - containerPort: 80
          env:
            - name: GET_HOSTS_FROM
              value: env
          resources:
            requests:
              cpu: 100m
              memory: 100Mi
---
apiVersion: v1
kind: Service
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30009
  selector:
    app: guestbook
    tier: frontend
```

---

## 🔍 Verification

Check all resources after applying manifests:

```bash
kubectl apply -f guestbook-backend.yaml
kubectl apply -f guestbook-frontend.yaml
kubectl get all -o wide
```
<br>  

<img width="1936" height="661" alt="Screenshot 2025-11-02 125134" src="https://github.com/user-attachments/assets/71bd8f57-93d9-4b53-bb87-80a6eba0086c" />  <br>

<br>  

✅ You should see:
- 3 frontend pods running  
- 1 Redis master  
- 2 Redis slaves  
- All services available (frontend NodePort → 30009)  

Then access the Guestbook app using or app button given in task:

```
http://<node-ip>:30009
```  

<img width="1368" height="225" alt="Screenshot 2025-11-02 125309" src="https://github.com/user-attachments/assets/31dd5e56-f5cc-4c94-9c72-babf17799c64" />  <br>


💬 You’ll see a form labeled **“Guestbook”** — where you can type a message and click **Submit**.  
The message gets stored in Redis and displayed on the page.

---

## 🧠 Key Learnings

- Learned how **multi-tier applications** can be deployed and scaled in Kubernetes.  
- Practiced **label-based service discovery** (`selector` & `matchLabels`).  
- Explored **NodePort Services** to expose apps externally.  
- Understood how **environment variables** (`GET_HOSTS_FROM`) influence container networking.  
- Experienced running both **stateful and stateless** workloads in one cluster.
- **Multi-tier orchestration:** Labels/selectors are foundational for wiring services to the right workloads.
- **Service abstraction:** ClusterIP and NodePort provide stable endpoints independent of pod lifecycles.
- **Read/write separation:** Master–slave topology enables scalable reads while maintaining a single write source.
- **Declarative ops:** Versioned YAML manifests make deployments reproducible and auditable.
- **Resource requests:** Explicit CPU/memory requests improve scheduling predictability.

---

## 🧭 Best Practices

1. Always use **labels and selectors** consistently for clarity and scaling.  
2. Use **resource requests/limits** to ensure fair scheduling.  
3. Expose services using **Ingress** for production environments instead of NodePort.  
4. Define **readiness probes** to ensure app stability before exposing to users.  
5. Keep YAML files modular — backend, frontend, configmaps, etc.  
6. **Pin images:** Use digests or versioned tags for reproducibility; avoid latest.  

---

## 🎯 Outcome

Deployed a functional guestbook stack:  

  - Redis Master (1) with internal Service for writes.
  - Redis Slaves (2) with internal Service for reads.
  - PHP Frontend (3) exposed via NodePort for external access.

This exercise demonstrates how Kubernetes makes multi-tier applications both declarative and scalable, even when mixing stateless frontends with stateful backends.

---

## 💭 Final Thoughts

This project beautifully demonstrates how multiple microservices communicate within Kubernetes.  
It’s not just about deploying containers — it’s about understanding **service discovery, scaling, and fault tolerance** in real-world DevOps systems.  

Day by day, this #100DaysOfDevOps challenge continues to strengthen the habit of **hands-on learning, experimentation, and community sharing**.  

Remember — the best way to master Kubernetes is to *break things, fix them, and document your learnings.*  
That’s how real engineers grow. 🚀  

---
