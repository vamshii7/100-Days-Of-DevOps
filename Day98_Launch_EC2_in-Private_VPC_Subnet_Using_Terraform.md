<div align=center>

# 🚀 Day 98: Launch EC2 Instance in a **Private VPC Subnet** Using Terraform

<img width="825" height="1079" alt="Screenshot 2025-12-03 123545" src="https://github.com/user-attachments/assets/686152e4-62b2-4907-92cc-a494cae429a0" />
<br>

</div>

------------------------------------------------------------------------

## 🧩 **Overview**

Goal: Build a secure, private AWS environment with Terraform:
 - Private VPC: datacenter-priv-vpc (10.0.0.0/16)
 - Private Subnet: datacenter-priv-subnet (10.0.1.0/24) - no auto-assign public IP
 - EC2 Instance: datacenter-priv-ec2 (t2.micro) deployed in the private subnet
 - Security Group: allows access only from inside the VPC CIDR (10.0.0.0/16)
 - Single main.tf file for resources; variables.tf and outputs.tf for variables & outputs
 - Working directory: /home/bob/terraform
   
> Open VS Code → EXPLORER → Right click → Open in Integrated Terminal to run commands there.

------------------------------------------------------------------------

## 📁 **Project Structure**

    .
    ├── main.tf
    ├── variables.tf
    └── outputs.tf

------------------------------------------------------------------------

## 🌐 **Terraform Files**

- `main.tf` - VPC, Subnet, Security Group, EC2 instance
- `variables.tf` - KKE_VPC_CIDR, KKE_SUBNET_CIDR variables (with defaults)
- `outputs.tf` - KKE_vpc_name, KKE_subnet_name, KKE_ec2_private outputs

 ## Complete Terraform code

 ### `main.tf`
 
 ```hcl
# VPC

resource "aws_vpc" "datacenter_priv_vpc" {
  cidr_block           = var.KKE_VPC_CIDR
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "datacenter-priv-vpc"
  }
}

# Subnet

resource "aws_subnet" "datacenter_priv_subnet" {
  vpc_id                  = aws_vpc.datacenter_priv_vpc.id
  cidr_block              = var.KKE_SUBNET_CIDR
  map_public_ip_on_launch = false

  tags = {
    Name = "datacenter-priv-subnet"
  }
}


# Security Group - Allow only VPC internal traffic

resource "aws_security_group" "datacenter_priv_sg" {
  name        = "datacenter-priv-sg"
  description = "Allow only internal VPC traffic"
  vpc_id      = aws_vpc.datacenter_priv_vpc.id

  # Allow ALL traffic only from VPC CIDR
  ingress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = [var.KKE_VPC_CIDR]
  }

  # Allow outbound anywhere
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "datacenter-priv-sg"
  }
}

# EC2 Instance

resource "aws_instance" "datacenter_priv_ec2" {
  ami                         = "ami-0c02fb55956c7d316" # Amazon Linux 2 (us-east-1)
  instance_type               = "t2.micro"
  subnet_id                   = aws_subnet.datacenter_priv_subnet.id
  vpc_security_group_ids      = [aws_security_group.datacenter_priv_sg.id]
  associate_public_ip_address = false
  
  tags = {
    Name = "datacenter-priv-ec2"
  
  }

  lifecycle {
    ignore_changes = [
      vpc_security_group_ids
    ]
  }
}
```

### Step-by-step explanation - each code block

- **VPC resource:** creates a VPC using the variable KKE_VPC_CIDR for the CIDR (default 10.0.0.0/16  
- **Subnet resource:** creates the private subnet in the VPC. `map_public_ip_on_launch = false` makes it a private subnet by default - instances launched here will not receive a public IP.  
- **Security Group:** allows all protocols/ports from inside the VPC CIDR only - meaning only other instances/resources inside 10.0.0.0/16 can reach this instance.  
    - Note (AWS rule): When protocol = "-1" (all protocols), from_port and to_port must both be 0. Using 0/65535 with -1 will produce an AWS API error.  
- **EC2 instance:** creates a t2.micro instance inside the private subnet, no public IP, attached to the security group that restricts ingress to the VPC.  
- **Why lifecycle.ignore_changes?**  
  - Terraform sometimes shows a repeated, one-line drift (~ vpc_security_group_ids = [ + "sg-..." ]) after apply - this is due to the SG being associated at the ENI level by AWS.
  - The ignore_changes here hides that known noise so terraform plan will report No changes (useful for lab/grading).
  - See the troubleshooting section for alternatives.


------------------------------------------------------------------------

### `variables.tf`

``` hcl
variable "KKE_VPC_CIDR" {
  description = "CIDR block for the VPC"
  default     = "10.0.0.0/16"
}

variable "KKE_SUBNET_CIDR" {
  description = "CIDR block for the subnet"
  default     = "10.0.1.0/24"
}
```

------------------------------------------------------------------------

### `outputs.tf`

``` hcl
output "KKE_vpc_name" {
  value = aws_vpc.datacenter_priv_vpc.tags["Name"]
}

output "KKE_subnet_name" {
  value = aws_subnet.datacenter_priv_subnet.tags["Name"]
}

output "KKE_ec2_private" {
  value = aws_instance.datacenter_priv_ec2.tags["Name"]
}

```
------------------------------------------------------------------------

## 🚀 Commands to Run

```bash
terraform init
terraform plan
terraform apply --auto-approve
```

---

## 🏁 Final Result --- Successful Apply

<img width="748" height="428" alt="Screenshot 2025-12-03 123558" src="https://github.com/user-attachments/assets/775b4e68-38d2-49b6-8a59-3e9153405189" />  <br>

<img width="927" height="170" alt="Screenshot 2025-12-03 124225" src="https://github.com/user-attachments/assets/f9414db5-78b4-4c84-b22a-6e682b972b1a" />


>After applying, terraform plan returns No changes.

------------------------------------------------------------------------

## Key learnings & best practices

- Private subnets are the default place for backend resources - they should not have public IPs. Use NAT or SSM for outbound access and management, not a public IP.
- Least privilege access: restrict Security Groups to minimal CIDRs and ports - you used VPC-only CIDR which is strong isolation.
- Avoid ignore_changes in production: it hides drift - prefer fixing root causes (explicit ENI attachment). Use ignore_changes judiciously in labs or when resources are intentionally managed outside Terraform.
- Use data-driven AMI lookups: AMI IDs are region-specific and time-variant. Prefer a data "aws_ami" lookup for portability.
- Remote state and locking: For teams, store state in S3 with DynamoDB locking to prevent race conditions.
- SSM for management: Use SSM Session Manager and instance profile for management without bastions or SSH.
- Tag everything: tags help cost reporting, audit and automation.

---
## Common issues & troubleshooting with fixes

### 1. Repeated ~ vpc_security_group_ids = [ + "sg-..." ] on terraform plan

   **Symptoms:** After apply, terraform plan still shows a one-line in-place update on the instance listing SGs.
   
   **Cause:** AWS attaches SGs to the instance network interface (ENI). AWS API sometimes returns SG membership differently (by ENI), producing a Terraform refresh/diff even though the configuration is correct.  

   **Fixes:**  

   - Option A (recommended for production): Use an explicit network_interface block in aws_instance so Terraform controls the ENI and its SGs. This removes the drift permanently.

     Example:  
   ```hcl
   resource "aws_instance" "datacenter_priv_ec2" {
   ami           = "ami-0c02fb55956c7d316"
   instance_type = "t2.micro"

   network_interface {
     device_index    = 0
     subnet_id       = aws_subnet.datacenter_priv_subnet.id
     security_groups = [aws_security_group.datacenter_priv_sg.id]
   }

   tags = { Name = "datacenter-priv-ec2" }
    }
   ```


- Option B (acceptable for labs/grading): Keep vpc_security_group_ids and add:

```hcl
lifecycle {
  ignore_changes = [ vpc_security_group_ids ]
}
```

*This hides the repeated diff. Downside: it also hides legitimate changes to SGs if done outside Terraform.*  

---  

### 2. `Error: from_port (0) and to_port (65535) must both be 0 to use the 'ALL' "-1" protocol!`  
  
  **Symptom:** Terraform/AWS rejects the security group rule.  
  
  **Cause:** AWS requires from_port = 0 and to_port = 0 when protocol = "-1".  
  
  **Fix:** Use from_port = 0 and to_port = 0 (as in the code above) or restrict to a specific protocol = "tcp" and appropriate ports.  

---

### 3. AMI not found / wrong region errors  

  **Symptom:** InvalidAMIID.NotFound: The image id '[ami-...]' does not exist.  
  
  **Cause:** AMI IDs are region-specific and can change.  
  
  **Fix:** Use a data source to dynamically find the latest Amazon Linux 2 AMI:

```hcl
data "aws_ami" "amazon_linux_2" {
  most_recent = true
  owners      = ["amazon"]
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}
# then
ami = data.aws_ami.amazon_linux_2.id
```
--- 

### 4. Cannot SSH to instance (expected)  
  **Cause:** Instance is in a private subnet and has no public IP.  
  
  **Fix / Access Patterns:**  Use a bastion (jump) host in a public subnet and SSH from there.  
  
  **Better:** attach an IAM role and use AWS SSM Session Manager to get shell access without any public IP or bastion.  

---  
### 5. Terraform state concerns / concurrency  
  
  **Tip:** Use S3 remote state + DynamoDB for locks in team settings to avoid corrupted state or race conditions.  
  
---

## 📝 Final Thoughts  

- Completing this task reinforces the foundational principles of secure and isolated AWS infrastructure using Terraform.  
- By launching an EC2 instance in a private VPC subnet, you achieved a setup where resources are protected from external internet access while still allowing secure intra-VPC communication.  
- This exercise not only strengthens Terraform skills but also reinforces security-first design principles for cloud-native infrastructure.

---

## 👨‍💻 Author  
**Vamshi Krishna**  
DevOps Engineer | DevOps & Kubernetes Enthusiast  
[Connect on LinkedIn](https://in.linkedin.com/in/vamshi7)  
> ⚙️ _Feel free to fork and contribute - PRs are welcome!_
---
