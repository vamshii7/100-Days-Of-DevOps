<div align=centre>

# 📅 Day 96: Create an EC2 Instance Using Terraform

The Nautilus DevOps team is continuing its step-by-step cloud migration strategy. Instead of shifting large workloads at once, the team divides the migration into smaller, manageable tasks. This ensures smooth transitions, predictable changes, and minimal disruption.  

<img width="778" height="678" alt="Screenshot 2025-12-01 081740" src="https://github.com/user-attachments/assets/f9985ae2-8fbe-4f10-87cb-ca89af6a1880" />

</div>
Today’s task focuses on provisioning a virtual machine in AWS using Terraform. EC2 instances form the foundation for most workloads in the cloud, making this an essential step toward a fully automated infrastructure.

---

## 📦 Task Requirements

Using Terraform, create an EC2 instance with the following specifications:

- **Instance name**: datacenter-ec2  
- **AMI**: ami-0c101f26f147fa7fd (Amazon Linux)  
- **Instance type**: t2.micro  
- **Create a new RSA key pair** named datacenter-kp  
- **Use the default AWS security group**  
- **Working directory**: `/home/bob/terraform`  
- Use a **single file named `main.tf`**  

---

## 🔑 Understanding AWS Key Pairs

AWS EC2 does not allow password logins by default. Instead, it uses **SSH key pairs** for secure, passwordless access.

### How key pairs work:

- The **public key** is stored in AWS  
- The **private key** stays with you locally  
- When connecting via SSH, your machine proves possession of the private key  
- AWS validates it against the stored public key  

This ensures that only the intended user can access the instance.

---

## 🔧 Step 1: Generate an SSH Key Pair

Before running Terraform, generate a new RSA key:

```bash
ssh-keygen -t rsa -b 4096 -f /home/bob/.ssh/id_rsa
```

This creates:

- /home/bob/.ssh/id_rsa → private key (never share)
- /home/bob/.ssh/id_rsa.pub → public key (safe to upload)

---

## 📄 Step 2: Create main.tf
```hcl
# Upload the public key as a new AWS key pair
resource "aws_key_pair" "datacenter_kp" {
  key_name   = "datacenter-kp"
  public_key = file("/home/bob/.ssh/id_rsa.pub")
}

# Fetch the default VPC
data "aws_vpc" "default" {
  default = true
}

# Fetch the default security group under the default VPC
data "aws_security_group" "default" {
  name   = "default"
  vpc_id = data.aws_vpc.default.id
}

# Create the EC2 instance
resource "aws_instance" "datacenter_ec2" {
  ami           = "ami-0c101f26f147fa7fd"
  instance_type = "t2.micro"
  key_name      = aws_key_pair.datacenter_kp.key_name

  vpc_security_group_ids = [
    data.aws_security_group.default.id
  ]

  tags = {
    Name = "datacenter-ec2"
  }
}
```
---

## ⚙️ Step 3: Run Terraform

```bash
terraform init
terraform plan
terraform apply --auto-approve
```

---

## 🔍 **Verify the State**

```sh
terraform state list
terraform state show <resource>
```
---

## 🧩 Common Issues and Troubleshooting

### ❌ Error: Access denied or SSH timeout

Check:
- Is port 22 allowed in the attached security group
- Are you using the correct private key
- Are permissions correct:
  - ```chmod 600 /home/bob/.ssh/id_rsa```

### ❌ InvalidKeyPair.Duplicate

AWS already has a key pair with the same name.  

Fix:
- Rename the key pair in Terraform
- Or delete the old key pair using AWS console or CLI

### ❌ AMI not found in region  
- Ensure the AMI exists in your region (us-east-1).
- AMI IDs are region specific.

### ❌ Terraform cannot find default VPC

Some accounts do not have a default VPC.
Fix:  
- Create one manually
- Or specify a custom VPC resource

### ❌ Permission denied for file read

Ensure Terraform can read your public key file:  
- ls -l /home/bob/.ssh/id_rsa.pub

---

## 💡 Key Takeaways

- Terraform makes cloud provisioning consistent and repeatable
- EC2 key pairs ensure secure, passwordless authentication
- Always secure your private keys
- Data sources help dynamically fetch existing AWS resources
- Tags help track resources and reduce operational chaos

---

## 🔗 Official Documentation

- Terraform AWS Provider: https://registry.terraform.io/providers/hashicorp/aws
- Terraform EC2 Instance: https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance
- Terraform Key Pair: https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/key_pair

---

## 🎯 Final Thoughts

Provisioning an EC2 instance with Terraform is one of the most essential skills in cloud automation. Today’s task strengthened my understanding of infrastructure as code, secure access, and AWS fundamentals. A solid foundation for bigger automation workflows ahead.

---

## 👨‍💻 Author  
**Vamshi Krishna**  
DevOps Engineer | DevOps & Kubernetes Enthusiast  
[Connect on LinkedIn](https://in.linkedin.com/in/vamshi7)  
> ⚙️ _Feel free to fork and contribute - PRs are welcome!_
---
