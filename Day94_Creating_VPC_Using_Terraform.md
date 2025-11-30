# 🚀 Day 94: Creating a VPC Using Terraform

<div align=center>
<img width="664" height="587" alt="Screenshot 2025-11-30 102919" src="https://github.com/user-attachments/assets/b49e1361-969d-4f54-8783-759e6ba3e5f6" />
</div>

<br>

Today’s task focused on learning how to create a VPC (Virtual Private Cloud) in AWS using Terraform. This is one of the foundational steps when working with Infrastructure as Code (IaC), and it helps beginners understand how cloud infrastructure can be provisioned automatically.

---

## To keep things simple and beginner-friendly, the goal was to:

- Initialize Terraform
- Create a simple main.tf file
- Provision a VPC named `datacenter-vpc` in the `us-east-1` region
- Assign any valid IPv4 CIDR block (`10.0.0.0/16` in this example)
- Apply the configuration and create the VPC successfully

---

## 📂 Directory Structure
Terraform directory was already created:

```
/home/bob/terraform
```

Inside this directory, we worked only with one file:

```
main.tf
```

---

## 📄 main.tf

```hcl
resource "aws_vpc" "main" {
  cidr_block       = "10.0.0.0/16"
  instance_tenancy = "default"

  tags = {
    Name = "datacenter-vpc"
  }
}
```

This file defines one resource:

- aws_vpc → Tells Terraform to create a VPC on AWS
- cidr_block → Defines the network range
- tags → Helps identify resources easily in AWS Console

---

## 🛠 Commands Used

### 1. Initialize Terraform
This downloads the required AWS provider:

```
terraform init
```

### 2. Preview the changes
This shows what Terraform plans to create:

```
terraform plan
```

### 3. Apply the configuration
This actually creates the VPC:

```
terraform apply -auto-approve
```

---

## 📸 Output Overview

- Terraform initialized successfully
  
  <img width="680" height="487" alt="Screenshot 2025-11-30 103427" src="https://github.com/user-attachments/assets/8f5676b1-71f5-4b42-96df-1ba5a6de7e98" />

- The plan showed that 1 resource will be created
  
  <img width="1080" height="657" alt="Screenshot 2025-11-30 103525" src="https://github.com/user-attachments/assets/b5d52f57-1676-4f5e-8100-75c3e12d9189" />

  
- After applying, the VPC was created with an ID such as:
  `vpc-54d039bb7042b7f4f`

  <img width="1020" height="665" alt="Screenshot 2025-11-30 103633" src="https://github.com/user-attachments/assets/2a92276a-9f59-4589-aef8-642569609359" />

  
- Status:
  **Apply complete! Resources: 1 added, 0 changed, 0 destroyed.**

---

## 🧠 Key Learnings

- Terraform makes infrastructure reproducible and automated
- The .tf file is readable and easy for beginners
- `terraform plan` is important to avoid mistakes
- Tags help identify cloud resources
- Small steps like creating a VPC prepare you for more complex cloud setups

---

## 📘 Official Documentation
Terraform AWS VPC Resource:  
https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc

---

## 🐛 Common Pitfalls, Errors & Troubleshooting Tips

### 1. **Provider Not Installed / terraform init Missing**
**Error:**
```
Provider registry.terraform.io/hashicorp/aws not found
```
**Fix:**  
Run `terraform init` before `plan` or `apply`.  
This downloads the AWS provider plugin.

---

### 2. **Missing AWS Credentials**
**Error:**
```
Error: No valid credential sources found
```
**Fix:**  
Make sure AWS credentials are configured:

```
aws configure
```
or export environment variables:

```
export AWS_ACCESS_KEY_ID=xxxx
export AWS_SECRET_ACCESS_KEY=xxxx
```

---

### 3. **Invalid or Overlapping CIDR Block**
**Error:**
```
error creating VPC: CIDR block is not valid
```
**Fix:**  
Use a valid CIDR block like `10.0.0.0/16`.  
Avoid overlaps with existing VPCs in the same AWS region.

---

### 4. **File Named Incorrectly**
Beginners often create `main.tf.txt` accidentally.

**Fix:**  
Ensure the file name is exactly:
```
main.tf
```

---

### 5. **Forgetting to Set the Region**
**Error:**
```
Error: Missing required argument: region
```
**Fix:**  
Add a provider block in the same file:

```hcl
provider "aws" {
  region = "us-east-1"
}
```

---

### 6. **Changes Not Applying**
**Symptom:** You updated the file, but Terraform shows *0 changes*.

**Fix:**
- Run `terraform fmt` to auto-fix formatting  
- Run `terraform validate`  
- Double-check indentation or misplaced `{ }`

---

### 7. **State File Corruption**
**Error:**
```
Error: state snapshot was created by a newer Terraform version
```
**Fix:**  
Upgrade Terraform to match:

```
terraform version
```

Or initialize again:

```
terraform init -upgrade
```

---

### 8. **VPC Already Exists**
If you manually created a VPC in AWS Console, Terraform will not know.

**Fix:**
Import the resource:

```
terraform import aws_vpc.main vpc-123456abcd
```

---

### 9. **Slow apply or timeout**
Creating VPCs can take time based on AWS API.

**Fix:**
Try again:

```
terraform apply
```
or check AWS status page.

---

### 10. **Tags Not Visible in AWS**
Sometimes tags don’t appear instantly.

**Fix:**  
Refresh AWS Console after a few seconds — caching delay is common.

---

## 👨‍💻 Author  
**Vamshi Krishna**  
DevOps Engineer | DevOps & Kubernetes Enthusiast  
[Connect on LinkedIn](https://in.linkedin.com/in/vamshi7)  
> ⚙️ _Feel free to fork and contribute — PRs are welcome!_
---

