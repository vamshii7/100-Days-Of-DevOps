<div align=center>

# 🚀 **Day 95: Creating a Security Group Using Terraform**

<img width="813" height="799" alt="Screenshot 2025-11-30 162455" src="https://github.com/user-attachments/assets/2c85ea90-d07d-4e12-af89-7eb3815ff21f" />  

<br>

</div>
As part of the Nautilus DevOps team's cloud migration strategy, the focus remains on breaking down large infrastructure tasks into smaller, manageable units. This approach ensures smooth adoption of AWS services with minimal disruption to ongoing workflows.  

Today’s objective is to use **Terraform** to create a **Security Group** inside the **default VPC** in **us-east-1**.

---

## ✅ **Task Requirements**

Using Terraform, create a security group with the following constraints:

1. **Name:** `nautilus-sg`  
2. **Description:** `Security group for Nautilus App Servers`  
3. **Inbound Rules:**  
   - **HTTP** → Port **80**, CIDR: `0.0.0.0/0`  
   - **SSH** → Port **22**, CIDR: `0.0.0.0/0`  
4. **Region:** `us-east-1`  
5. **VPC:** Must use the **default VPC**  
6. All configuration must be placed in **main.tf** located at `/home/bob/terraform`

---

## 📁 **Directory Structure**
```
/home/bob/terraform
 └── main.tf
 └── provider.tf
```

---

## 🧩 **main.tf - Final Terraform Configuration**

```hcl
# Fetch the default VPC ID
data "aws_vpc" "default" {
  default = true
}

# Create the security group
resource "aws_security_group" "nautilus_sg" {
  name        = "nautilus-sg"
  description = "Security group for Nautilus App Servers"
  vpc_id      = data.aws_vpc.default.id

  # Inbound rule for HTTP (port 80)
  ingress {
    description = "Allow HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Inbound rule for SSH (port 22)
  ingress {
    description = "Allow SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Outbound rule (allow all traffic)
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "nautilus-sg"
  }
}
```

---

## ▶️ **Terraform Commands to Run**

```sh
terraform init
terraform plan
terraform apply -auto-approve
```  

<img width="1069" height="967" alt="Screenshot 2025-11-30 162447" src="https://github.com/user-attachments/assets/c5ca2fa0-0a93-4fbd-a87e-5631f2ff46dd" />
<img width="1020" height="970" alt="Screenshot 2025-11-30 162539" src="https://github.com/user-attachments/assets/10d38520-7cf5-48db-939b-1fa5d492f3cb" />

---

## 🔍 **Verify the State**

```sh
terraform state list
terraform state show aws_security_group.nautilus_sg
```  

<img width="785" height="507" alt="Screenshot 2025-11-30 163138" src="https://github.com/user-attachments/assets/3cb99057-53c1-4409-b6d1-129b289f4762" />
<img width="836" height="734" alt="Screenshot 2025-11-30 163214" src="https://github.com/user-attachments/assets/7a9d2d90-8997-4b2c-af9e-c0ab6f587a07" />


---

## ⚠️ Common Pitfalls & Troubleshooting

### ❌ AWS credentials not configured  
Fix:
```
aws configure
```

### ❌ Default VPC not found  
Fix: Create a VPC or specify VPC ID manually.

### ❌ Region mismatch  
Fix:
```
aws configure get region
```

### ❌ Security Group name already exists  
Fix:
```
terraform import aws_security_group.nautilus_sg sg-123456
```

### ❌ Typo in Terraform blocks  
Fix: Validate with latest AWS provider documentation.

---

## 🎯 Outcome

You learned to:

- Use the AWS provider with Terraform  
- Query default VPC using data sources  
- Create a security group with multiple ingress rules  
- Inspect state information  
- Troubleshoot AWS/Terraform issues effectively  

---

## 👨‍💻 Author  
**Vamshi Krishna**  
DevOps Engineer | DevOps & Kubernetes Enthusiast  
[Connect on LinkedIn](https://in.linkedin.com/in/vamshi7)  
> ⚙️ _Feel free to fork and contribute - PRs are welcome!_
---
