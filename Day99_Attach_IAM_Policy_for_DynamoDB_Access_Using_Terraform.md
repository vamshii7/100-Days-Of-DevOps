# **Day 99: Attach IAM Policy for DynamoDB Access Using Terraform**

This task focuses on creating a secure AWS environment using Terraform
by provisioning a DynamoDB table and attaching a restricted IAM policy
to an IAM role. The policy enforces **read‑only** access to ensure the
principle of least privilege.

------------------------------------------------------------------------

## 🚀 **Task Overview**

You are required to:

1.  Create a DynamoDB table named `devops-table`.
2.  Create an IAM Role `devops-role`.
3.  Create an IAM Policy `devops-readonly-policy` granting:
    -   `GetItem`
    -   `Scan`
    -   `Query`
4.  Attach the policy to the IAM role.
5.  Define inputs using **variables.tf**, **terraform.tfvars**.
6.  Output essential resource names via **outputs.tf**.
7.  Ensure `terraform plan` shows **No changes** before submission.

------------------------------------------------------------------------

# 📌 **main.tf**

``` hcl
# 1. Create DynamoDB Table
resource "aws_dynamodb_table" "devops_table" {
  name         = var.KKE_TABLE_NAME
  billing_mode = "PAY_PER_REQUEST"

  hash_key = "id"

  attribute {
    name = "id"
    type = "S"
  }
}

# 2. Create IAM Role
resource "aws_iam_role" "devops_role" {
  name = var.KKE_ROLE_NAME

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "ec2.amazonaws.com"
      }
    }]
  })
}

# 3. Create IAM Policy (Read-Only DynamoDB)
resource "aws_iam_policy" "devops_readonly_policy" {
  name        = var.KKE_POLICY_NAME
  description = "Read-only access to specific DynamoDB table"

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "dynamodb:GetItem",
          "dynamodb:Scan",
          "dynamodb:Query"
        ]
        Resource = aws_dynamodb_table.devops_table.arn
      }
    ]
  })
}

# 4. Attach Policy to Role
resource "aws_iam_role_policy_attachment" "attach_readonly_policy" {
  role       = aws_iam_role.devops_role.name
  policy_arn = aws_iam_policy.devops_readonly_policy.arn
}
```

------------------------------------------------------------------------

## 📝 **Explanation of Important Code Snippets**

### **1. DynamoDB Table Creation**

-   `PAY_PER_REQUEST` billing mode eliminates manual capacity planning.
-   A single `id` attribute keeps configuration minimal.
-   Ensures a lightweight and cost‑effective structure.

### **2. IAM Role with Assume Role Policy**

-   EC2 is allowed to assume the role.
-   Identity federation and trust relationships follow AWS best
    practices.
-   Encapsulated using `jsonencode` for safety and indentation
    consistency.

### **3. Read‑Only IAM Policy**

-   Principle of Least Privilege:
    -   Only required actions: `GetItem`, `Scan`, `Query`.
    -   Only on the **specific table ARN**.
-   Helps prevent privilege escalation and protects data integrity.

### **4. Role‑Policy Attachment**

-   Separates role creation from policy attachment → clean, reusable
    Terraform structure.
-   Ensures modularity and clarity.

------------------------------------------------------------------------

# 📌 **variables.tf**

``` hcl
variable "KKE_TABLE_NAME" {
  description = "Name of the DynamoDB table"
  type        = string
}

variable "KKE_ROLE_NAME" {
  description = "Name of the IAM role"
  type        = string
}

variable "KKE_POLICY_NAME" {
  description = "Name of the IAM policy"
  type        = string
}
```

------------------------------------------------------------------------

# 📌 **outputs.tf**

``` hcl
output "kke_dynamodb_table" {
  value = aws_dynamodb_table.devops_table.name
}

output "kke_iam_role_name" {
  value = aws_iam_role.devops_role.name
}

output "kke_iam_policy_name" {
  value = aws_iam_policy.devops_readonly_policy.name
}
```

------------------------------------------------------------------------

# 📌 **terraform.tfvars**

``` hcl
KKE_TABLE_NAME  = "devops-table"
KKE_ROLE_NAME   = "devops-role"
KKE_POLICY_NAME = "devops-readonly-policy"
```

------------------------------------------------------------------------

# 🧠 **Key Takeaways**

### ✔️ **1. Least Privilege Access**

Always restrict IAM policies to the absolute minimum required actions.

### ✔️ **2. Resource‑Specific Permissions**

Hard‑coding or allowing `*` access is dangerous → always scope
permissions to ARNs.

### ✔️ **3. Using jsonencode**

Prevents formatting mistakes in JSON‑based policies.

### ✔️ **4. Separation of Config**

main.tf → resources\
variables.tf → inputs\
outputs.tf → resource exposure\
terraform.tfvars → environment values

### ✔️ **5. Reproducibility**

Terraform ensures consistent provisioning across environments.

------------------------------------------------------------------------

# ⚠️ **Common Issues & Troubleshooting**

  -----------------------------------------------------------------------------------------
  Issue                       Cause                      Fix
  --------------------------- -------------------------- ----------------------------------
  `MalformedPolicyDocument`   JSON formatting errors     Use `jsonencode()`

  IAM role not assumable      Wrong AWS service in       Ensure
                              principal                  `"Service": "ec2.amazonaws.com"`

  `AccessDeniedException`     Missing actions in policy  Add `GetItem`, `Scan`, `Query`

  Terraform doesn't detect    State mismatch             Run `terraform refresh`
  changes                                                

  Table creation failure      Duplicate table names      Update `KKE_TABLE_NAME`
  -----------------------------------------------------------------------------------------

------------------------------------------------------------------------

# 🎉 **Final Thoughts & Achievement**

Today's task reinforces core DevOps & cloud fundamentals:

-   Writing clean Terraform IaC 🏗️\
-   Understanding IAM security 🔐\
-   Implementing least privilege access 📉\
-   Managing AWS DynamoDB using automation ⚡\
-   Building production‑grade infra with version control readiness 📁

You've successfully automated secure DynamoDB access with Terraform ---
a **high‑value real‑world DevOps skill**.

Keep going! 🚀

------------------------------------------------------------------------

**Created for GitHub --
Day99_Attach_IAM_Policy_for_DynamoDB_Access_Using_Terraform.md**
