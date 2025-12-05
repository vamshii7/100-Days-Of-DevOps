# Day 100: Create and Configure CloudWatch Alarm Using Terraform

## 🚀 Task Overview  
<div align=center>
  <img width="734" height="931" alt="Screenshot 2025-12-05 124811" src="https://github.com/user-attachments/assets/10a97c68-a37a-47ba-bc68-5a812c7c8dc5" />

</div>

The Nautilus DevOps team needs to deploy an EC2 instance and configure a CloudWatch alarm to monitor CPU utilization. The alarm should trigger when CPU usage exceeds **90%** for one **5-minute** period and notify an existing SNS topic (`datacenter-sns-topic`).

---

## 📁 **main.tf**
```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_sns_topic" "sns_topic" {
  name = "datacenter-sns-topic"
}

# 1. EC2 Instance
resource "aws_instance" "datacenter_ec2" {
  ami           = "ami-0c02fb55956c7d316"
  instance_type = "t2.micro"

  tags = {
    Name = "datacenter-ec2"
  }
}

# 2. CloudWatch Alarm
resource "aws_cloudwatch_metric_alarm" "datacenter_alarm" {
  alarm_name          = "datacenter-alarm"
  comparison_operator = "GreaterThanOrEqualToThreshold"
  evaluation_periods  = 1
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = 300
  statistic           = "Average"
  threshold           = 90

  alarm_description = "Triggers when CPU Utilization exceeds 90% for one 5-minute period."

  dimensions = {
    InstanceId = aws_instance.datacenter_ec2.id
  }

  alarm_actions = [
    aws_sns_topic.sns_topic.arn
  ]
}
```

---

## 📁 **outputs.tf**
```hcl
output "KKE_instance_name" {
  value = aws_instance.datacenter_ec2.tags["Name"]
}

output "KKE_alarm_name" {
  value = aws_cloudwatch_metric_alarm.datacenter_alarm.alarm_name
}
```

---

## 🖥️ Terraform Commands

### Initialize:
```bash
terraform init
```

### Validate:
```bash
terraform validate
```

### Plan:
```bash
terraform plan
```

### Apply:
```bash
terraform apply -auto-approve
```

### 🖼️ Expected Output  

<img width="948" height="460" alt="Screenshot 2025-12-05 124753" src="https://github.com/user-attachments/assets/8ee1ced4-ebae-42dc-b00b-53eff77b4880" />  

---

## 🌱 Why this matters in the DevOps journey

- This hands-on builds the foundations of infrastructure observability, a pillar of DevOps.
- Learning to detect, alert, and respond to system metrics is essential for:
    - High availability
    - Incident prevention
    - Faster recovery
    - Production readiness  
- Every system needs guardrails - and today’s exercise reinforced how to build those guardrails using IaC.

---

## 🧠 Key Learnings
- Provisioning an EC2 instance with Terraform  
- Creating CloudWatch alarms for proactive monitoring  
- Binding alarms to SNS notifications  
- Using CloudWatch metric dimensions  
- Using Terraform outputs to expose important resource details  

---

## ⭐ Best Practices
- Always tag your AWS resources  
- Set meaningful alarm thresholds to avoid alert fatigue  
- Validate before applying changes  
- Use SNS topics for centralized alert routing  
- Reuse outputs wherever possible  

---

# 🎉 *Congratulations on completing Day 100 of your DevOps Journey!*  

---

## 👨‍💻 Author  
**Vamshi Krishna**  
DevOps Engineer | DevOps & Kubernetes Enthusiast  
[Connect on LinkedIn](https://in.linkedin.com/in/vamshi7)  
> ⚙️ _Feel free to fork and contribute - PRs are welcome!_
---
