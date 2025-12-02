# Day 97: Create IAM Policy Using Terraform

Identity and Access Management (IAM) is one of the foundational services to configure when working with AWS. It enables secure control over who can access specific resources and what actions they can perform. In this task, we focus on creating a custom IAM policy using Terraform - an essential skill for automating cloud security configurations.

---

## 🎯 Task Objective

<img width="717" height="385" alt="Screenshot 2025-12-02 113306" src="https://github.com/user-attachments/assets/89789950-f459-4c68-a482-1ad0c24df12c" />

<br>

Create an IAM policy in the **us-east-1** region using **Terraform**, granting **read-only access to Amazon EC2** resources.  
This includes permission to **view instances, AMIs, snapshots, volumes**, and other EC2 metadata.

The Terraform working directory is: `/home/bob/terraform`

Create only the `main.tf` file (no additional `.tf` files).

## 📌 main.tf

```hcl
# Create IAM Policy for EC2 Read-Only Access
resource "aws_iam_policy" "iampolicy_rose" {
  name        = "iampolicy_rose"
  description = "IAM policy for EC2 read-only access"

  policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "EC2ReadOnlyAccess",
      "Effect": "Allow",
      "Action": [
        "ec2:Describe*"
      ],
      "Resource": "*"
    }
  ]
}
EOF
}
```

## ⚙️ Explanation

- Provider: AWS in region us-east-1
- Policy Name: iampolicy_rose
- Effect: Allow
- Action: ec2:Describe* → Grants read-only permissions for all EC2 Describe APIs
- Resource: * → Applies to all EC2 resources
- This allows users to view:
    - EC2 instances
    - AMIs
    - Snapshots
    - Volumes
    - Security groups
    - Other read-only metadata
- But cannot create, modify, or delete anything.

---

## 🚀 Commands to Run

```bash
terraform init
terraform plan
terraform apply --auto-approve
```

---

✅ Expected Result

- A new IAM policy is created in AWS IAM.
- Grants read-only access to the EC2 console.
- Users can view EC2 resources but cannot perform write or destructive operations.

<img width="1034" height="694" alt="Screenshot 2025-12-02 114352" src="https://github.com/user-attachments/assets/5b69d6ad-25c9-4d5a-8b98-e53af75762eb" />

---

## 🎉 Outcome

In this task, we have created a Terraform-managed IAM policy that grants read-only access to Amazon EC2 resources. Understanding IAM, JSON policies, and infrastructure-as-code is essential when building secure cloud environments.

---

## 📝 Common Pitfalls & Troubleshooting
### ❗ 1. Incorrect IAM Policy JSON Formatting  
  - Terraform may fail with a parsing error.
    - Fix: Validate JSON using jq or an online formatter.

### ❗ 2. Policy Name Already Exists  
  - AWS prevents duplicate policy names.  
    - Fix: Rename the policy or delete the old one.

### ❗ 3. Missing or Incorrect Provider Configuration  
  - If the AWS region isn’t set, Terraform may deploy to the wrong region.
    - Fix: Ensure provider configuration matches your intended region.  

### ❗ 4. Insufficient IAM Permissions  
  - Terraform needs IAM privileges to create policies.
    - Fix: Ensure your IAM role/user has:  
      - iam:CreatePolicy  
      - iam:ListPolicies
      - iam:GetPolicy

### ❗ 5. JSON Heredoc Not Closed Properly  
  - If the EOF blocks break, Terraform won’t parse the policy.  
    * Fix:  
      - Ensure EOF is not indented  
      - Must start and end on its own line
---

## 🛡️ Best Practices for IAM & Terraform

### 🔐 1. Follow the Principle of Least Privilege
Always grant only the permissions a user or service strictly needs.  
Avoid using overly broad policies like `"*"` actions or full admin access.  

---

### 📦 2. Use Descriptive Policy Names and Descriptions
Clear naming helps with long-term maintainability.  
Example: `ec2_readonly_policy` clearly describes its purpose.

---

### 🗂️ 3. Store IAM Policies in Separate JSON Files (When Policies Are Large)
For small policies, inline text is fine.  
For complex policies, store them in `policy.json` and reference using:

```hcl
policy = file("policy.json")
```
This keeps Terraform files clean and readable.  

---

### 🔄 4. Enable Version Control for All IAM Policies  
Never edit IAM policies manually from AWS console if Terraform manages them.
Use Git to track changes, reviews, and rollbacks.  

---

### 🚫 5. Avoid Hardcoding Sensitive Information

Never store secrets, tokens, or private keys inside Terraform code.
Use:  
 - AWS SSM Parameter Store  
 - Secrets Manager  
 - Environment variables

---

### 🧪 6. Validate JSON Policies Frequently

Malformed JSON can break Terraform plans, especially with heredocs.

Check using: `jq . policy.json`  

---

### 🧩 7. Keep IAM Policies Modular and Reusable

Instead of creating many similar policies, make general reusable ones.

Example:  
   - EC2 read-only policy  
   - S3 read-only policy  
   - CloudWatch logs basic access policy  
   - Mix and match for roles and groups.

---

### 🚀 8. Use Terraform State Locking

When working in teams, enable state locking using:  
   - S3 backend  
   - DynamoDB lock table 
   - Prevents accidental overwrites of IAM or other resources.

---

### 🧑‍💻 9. Don’t Assign Policies Directly to Users

Instead:  
   - Create groups  
   - Attach policies to groups  
   - Add users to groups  
   - This avoids policy sprawl and simplifies access management.

---

### 🔍 10. Periodically Review IAM Policies

Cloud environments change over time.
Review policies to ensure they match current requirements and compliance rules.

---

## 👨‍💻 Author  
**Vamshi Krishna**  
DevOps Engineer | DevOps & Kubernetes Enthusiast  
[Connect on LinkedIn](https://in.linkedin.com/in/vamshi7)  
> ⚙️ _Feel free to fork and contribute - PRs are welcome!_
---
