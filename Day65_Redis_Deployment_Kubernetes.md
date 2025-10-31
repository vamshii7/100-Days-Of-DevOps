# 🚀 Day 65 of #100DaysOfDevOps

### 🧠 Focus: Deploying Redis on Kubernetes with ConfigMap and Deployment

---

## 📘Task Overview  

<img width="983" height="802" alt="Screenshot 2025-10-31 110618" src="https://github.com/user-attachments/assets/4ee5bddb-3b2f-4318-bee0-fa3286326868" />  

<br>  

Today, we deployed a **Redis** instance on Kubernetes using a `ConfigMap` to manage Redis configuration and a `Deployment` to handle pod lifecycle.  
This setup demonstrates how to externalize configurations and mount them inside containers — a key DevOps practice for flexibility and maintainability.

---

## ⚙️ Step 1: Create the ConfigMap

**ConfigMap Manifest:**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-redis-config
data:
  redis-config: |
    maxmemory 2mb
```

**Apply the manifest:**

```bash
kubectl apply -f redis-deploy.yaml
```

**Verify ConfigMap:**

```bash
kubectl get cm
kubectl describe cm my-redis-config
```

<img width="351" height="400" alt="Screenshot 2025-10-31 113306" src="https://github.com/user-attachments/assets/d0ae5424-267d-43ae-a175-7fe8167b6570" />  

<br> 

The ConfigMap was successfully created containing our Redis configuration (`maxmemory 2mb`).

---

## ⚙️ Step 2: Create Redis Deployment

**Deployment Manifest:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
        - name: redis-container
          image: redis:alpine
          resources:
            requests:
              cpu: "1"
          volumeMounts:
            - name: data
              mountPath: /redis-master-data
            - name: redis-config
              mountPath: /redis-master
          ports:
            - containerPort: 6379
      volumes:
        - name: data
          emptyDir: {}
        - name: redis-config
          configMap:
            name: my-redis-config
            items:
              - key: redis-config
                path: redis-config
```

**Apply the manifest:**

```bash
kubectl apply -f redis-deploy.yaml
```

**Check Deployment status:**

```bash
kubectl get all -n default
```  

<img width="567" height="292" alt="Screenshot 2025-10-31 113216" src="https://github.com/user-attachments/assets/61723eb9-9d87-455d-a7ef-e4021075d62e" />  


```bash
kubectl describe deploy redis-deployment
```  
<img width="807" height="854" alt="Screenshot 2025-10-31 113228" src="https://github.com/user-attachments/assets/280fbe35-598b-4ac7-addd-2d6f04bd8865" />  

<br>   

A new deployment and pod were successfully created. The pod was in **Running** state with Redis container active.

---

## 🧩 Step 3: Verify Mounted Config and Volumes

You can exec into the pod to verify the mounted files:

```bash
kubectl exec -it $(kubectl get pod -l app=redis -o jsonpath='{.items[0].metadata.name}') -- /bin/sh
ls /redis-master
cat /redis-master/redis-config
```

This confirms the `ConfigMap` is mounted properly inside the container.

---

## 💡 Key Learnings

- **ConfigMaps** are ideal for managing non-sensitive configuration outside containers.  
- **Deployments** manage replica scaling and updates seamlessly.  
- **emptyDir** volumes are **ephemeral** (lost after pod restart). Use **PersistentVolumes** for production.  
- Always pair resource `requests` with `limits` for predictable scheduling.  
- Validate configurations before rolling updates.

---

## 🧠 Best Practices for Redis Deployment on Kubernetes

### 🔹 ConfigMap & Secrets  
1. **Separate configurations from code**  
   Store environment configs in `ConfigMaps`, and credentials (like passwords) in `Secrets`.

2. **Version control your manifests, but never commit secrets**  
   Keep your YAML manifests in Git, but do not check in real credentials or access tokens.

3. **Mount ConfigMaps as files when required**  
   When the app expects configuration files (like Redis), mount the ConfigMap as a file instead of injecting values as environment variables.  

---

### 🔹 Container & Resource Management  
4. **Pin image versions**  
   Use explicit image tags (e.g., `redis:7.0-alpine`) instead of `:latest` to ensure reproducibility.

5. **Always define resource requests and limits**  
   Specify CPU and memory requests/limits to ensure proper Pod scheduling and avoid noisy-neighbour issues.  

---

## 🏁 Final Thoughts

Today’s session bridged the concepts of **config management** and **application deployment** in Kubernetes.  
Redis serves as an excellent use case to demonstrate how Kubernetes abstracts configuration and runtime environments for any stateful application.

---  

**Task Completed Successfully 🎉**  

<img width="1819" height="979" alt="Screenshot 2025-10-31 113438" src="https://github.com/user-attachments/assets/666cc26b-5413-4d6f-bc00-a7728be3d732" />  

---

