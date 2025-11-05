# 🚀 Kubernetes Level-3 Task: Configure LAMP Stack Application  

<img width="967" height="1227" alt="Screenshot 2025-11-05 093024" src="https://github.com/user-attachments/assets/ce9f7f30-cc5f-4160-80c0-3a565bf37ff0" />  <br>


This task involves deploying a **LAMP stack application** (Linux, Apache, MySQL, PHP) on **Kubernetes** using **ConfigMaps**, **Secrets**, and **Environment Variables** to manage configuration and credentials securely.  

This guide demonstrates how to deploy a **LAMP** stack on a Kubernetes cluster using ConfigMaps, Secrets, and environment variables.  

---

## 🧠 Task Overview

In this task, you will:

- Create ConfigMaps and Secrets for PHP and MySQL configurations  
- Deploy a LAMP stack using a multi-container Pod (Apache + MySQL)  
- Expose the application using a NodePort service  
- Validate connectivity between PHP and MySQL using environment variables

---

## ⚙️ Step 1: Create ConfigMap

Create a ConfigMap named `php-config` for PHP configuration.  
`php-config.yaml`  

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: php-config
data:
  php.ini: |
    variables_order = "EGPCS"
```

Apply it:
```bash
kubectl apply -f php-config.yaml
```

---

## 🔐 Step 2: Create Secrets  
`mysql-secrets.yaml`  

```yaml
### Secret 1 – MySQL Root Password

apiVersion: v1
kind: Secret
metadata:
  name: mysql-root-pass
type: Opaque
stringData:
  password: j@rv!s

---

### Secret 2 – MySQL User and Password

apiVersion: v1
kind: Secret
metadata:
  name: mysql-user-pass
type: Opaque
stringData:
  username: vamshi
  password: j@rv!s

---

### Secret 3 – MySQL Database Name

apiVersion: v1
kind: Secret
metadata:
  name: mysql-db-url
type: Opaque
stringData:
  database: mydb
```

Apply secrets:
```bash
kubectl apply -f mysql-secrets.yaml
```

---

## 🏗️ Step 3: Define the Deployment (lamp-deploy.yaml)

This deployment contains **two containers** – one for PHP+Apache and one for MySQL.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: lamp-wp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: lamp
  template:
    metadata:
      labels:
        app: lamp
    spec:
      containers:
      - name: httpd-php-container
        image: webdevops/php-apache:alpine-3-php7
        ports:
        - containerPort: 80
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
          - name: MYSQL_HOST
            value: mysql-service
        volumeMounts:
          - name: php-config
            mountPath: /opt/docker/etc/php/php.ini
            subPath: php.ini
      - name: mysql-container
        image: mysql:5.6
        ports:
        - containerPort: 3306
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
      - name: php-config
        configMap:
          name: php-config
```

Apply it:
```bash
kubectl apply -f lamp-deploy.yaml
```

---

## 🌐 Step 4: Create Services  

`lamp-services.yaml`  

```yaml
# MySQL Service

apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  selector:
    app: lamp
  ports:
  - port: 3306
    targetPort: 3306
---

# Web Application Service (NodePort)

apiVersion: v1
kind: Service
metadata:
  name: lamp-service
spec:
  type: NodePort
  selector:
    app: lamp
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30008
```

Apply:
```bash
kubectl apply -f lamp-services.yaml
```

---

## 🔍 Step 5: Apply and Verify Resources

Check deployments, pods, and services:

```bash
kubectl get all
```

📸 *After applying manifests*  <br>  
<img width="643" height="420" alt="Screenshot 2025-11-05 103955" src="https://github.com/user-attachments/assets/b74decec-4da0-4559-b877-1e6daf53d383" />


---

## 🧾 Step 6: Modify PHP Application (index.php)

We already have `/tmp/index.php` on the jump host. Copy it into the container and modify it to use environment variables.

```bash
kubectl cp /tmp/index.php lamp-wp-<pod-name>:/app -c httpd-php-container
kubectl exec -it lamp-wp-<pod-name> -c httpd-php-container -- sh
vi /app/index.php
```  
<img width="806" height="313" alt="Screenshot 2025-11-05 104010" src="https://github.com/user-attachments/assets/6246e72e-a014-4005-a89b-b072660da001" />  <br>

Replace contents with the below code:  

```php
<?php
// Fetch MySQL connection details from environment variables
$dbname = getenv('MYSQL_DATABASE');
$dbuser = getenv('MYSQL_USER');
$dbpass = getenv('MYSQL_PASSWORD');
$dbhost = getenv('MYSQL_HOST');

// Create connection
$conn = mysqli_connect($dbhost, $dbuser, $dbpass, $dbname);

// Check connection
if (!$conn) {
  die("Connection failed: " . mysqli_connect_error());
}

// Run a simple query to verify DB access
$test_query = "SHOW TABLES";
$result = mysqli_query($conn, $test_query);

if (!$result) {
  echo "Connected to MySQL but could not query database '$dbname'.<br>";
  echo "Error: " . mysqli_error($conn);
} else {
  echo "Connected successfully";
}

mysqli_close($conn);
?>
```

📸 *Editing and verifying index.php inside container*    

<img width="1160" height="1124" alt="image" src="https://github.com/user-attachments/assets/6abf15da-917a-4ed3-8e15-4ea06a01d964" />
  
<br>

---

## 🔎 Step 7: Check Environment Variables inside Pod  

```bash
kubectl exec -it lamp-wp-<pod-name> -c httpd-php-container -- env | grep MYSQL
```

📸 *Verifying environment variables*    

<img width="796" height="391" alt="image" src="https://github.com/user-attachments/assets/532cf9b7-5639-4e19-abc2-38df52d78516" />  <br>

---

## ✅ Step 8: Verify MySQL Connection

Access the application from your browser using:

```
http://<node-ip>:30008
```

You should see:
```
Connected successfully  
```  

<img width="543" height="87" alt="Screenshot 2025-11-05 104310" src="https://github.com/user-attachments/assets/f93167ae-2852-421f-9e2b-1cbd276a3cff" />  

---

## 🧩 Key Learnings / Takeaways

- ConfigMaps are ideal for non-sensitive configurations.
- Secrets securely store sensitive credentials.
- Multiple containers can share environment variables via Services.
- NodePort exposes applications externally on a specific port.

---

## 🧭 Best Practices

- Use `envFrom` for cleaner environment variable injection (optional).
- Store Secrets in external vaults (like HashiCorp Vault or Azure Key Vault) for production.
- Always define `liveness` and `readiness` probes for each container.
- Use Persistent Volumes for database data instead of ephemeral storage.

---

## 💡 Final Thoughts

This task demonstrated how to set up a **multi-container LAMP stack** using **ConfigMaps, Secrets, and NodePort Services** in Kubernetes.  
The setup validates a real-world PHP–MySQL integration scenario under containerized orchestration.

---
