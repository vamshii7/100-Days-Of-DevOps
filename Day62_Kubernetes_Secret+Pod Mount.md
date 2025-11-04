# 🔐 Kubernetes Secret + Pod Mount — Hands-On Solution

## 🧠 Objective

We need to:

1. Create a **generic Secret** named `news` using an existing file `/opt/news.txt` (contains password/license number).  
2. Create a **Pod** named `secret-datacenter` with:
   - Container name: `secret-container-datacenter`
   - Image: `debian:latest`
   - Command: `sleep` to keep container running
   - Mounted Secret under `/opt/games`

---

## 🧩 Step 1: Create the Secret from File

The file `/opt/news.txt` already exists on the jump host.

Run the following command to create the Secret:

```bash
kubectl create secret generic news --from-file=/opt/news.txt
```

This command creates:
- **Secret Name**: `news`
- **Key**: `news.txt`
- **Value**: (content of `/opt/news.txt`)

### 🔍 Verify the Secret

```bash
kubectl get secrets
kubectl describe secret news
```

---

## 🧱 Step 2: Create the Pod Definition

Save the below manifest as `secret-datacenter.yaml`.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-datacenter
spec:
  containers:
  - name: secret-container-datacenter
    image: debian:latest
    command: ["sleep", "3600"]   # Keeps container running
    volumeMounts:
    - name: secret-volume
      mountPath: "/opt/games"
  volumes:
  - name: secret-volume
    secret:
      secretName: news
```

---

## 🚀 Step 3: Apply and Verify

Apply the manifest:

```bash
kubectl apply -f secret-datacenter.yaml
```

Check Pod status:

```bash
kubectl get pods
```

✅ The Pod should reach the `Running` state.

---

## 🕵️ Step 4: Inspect Mounted Secret Inside Container

Exec into the running container:

```bash
kubectl exec -it secret-datacenter -- bash
```

Inside the container:

```bash
ls /opt/games
cat /opt/games/news.txt
```

You should see the secret content (same as in `/opt/news.txt`).

---

## ✅ Validation Checklist

| Checkpoint | Command | Expected Output |
|-------------|----------|----------------|
| Secret created | `kubectl get secrets` | `news` listed |
| Pod running | `kubectl get pods` | STATUS = Running |
| Secret mounted | `ls /opt/games` | shows `news.txt` |
| Secret readable | `cat /opt/games/news.txt` | matches `/opt/news.txt` |

---

## 🧩 Notes & Best Practices

- 🔐 **Secrets are base64-encoded**, not encrypted — use encryption at rest for production clusters.  
- 📁 Secrets must exist **in the same namespace** as the consuming Pod.  
- ⚙️ For sensitive workloads, use external Secret stores like **AWS Secrets Manager**, **HashiCorp Vault**, or **Azure Key Vault**.  
- 🧹 Never commit actual secret files or encoded YAMLs to GitHub.

---

## 🧰 Useful Commands Recap

```bash
kubectl create secret generic news --from-file=/opt/news.txt
kubectl apply -f secret-datacenter.yaml
kubectl exec -it secret-datacenter -- bash
kubectl get secrets,pods
```

--- 

💡 *This task demonstrates secure configuration delivery via Kubernetes Secrets, ensuring credentials remain decoupled from application logic.*
