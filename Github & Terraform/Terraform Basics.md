# **Terraform Basic Commands**  
**Boritrade, LLC**  
**Last Updated:** January 30, 2025  

## **Basic Terraform Commands**

### 1. **Initialize Terraform**
   ```
   terraform init
   ```
   - **What it does:** Initializes the Terraform working directory, downloads necessary provider plugins.
   - **When to use:** Run it first before applying any Terraform configuration.

### 2. **Format Terraform Code**
   ```
   terraform fmt
   ```
   - **What it does:** Formats Terraform code to follow standard formatting guidelines.
   - **When to use:** Run it before committing code to maintain consistency.

### 3. **Validate Configuration**
   ```
   terraform validate
   ```
   - **What it does:** Checks the configuration files for syntax errors.
   - **When to use:** Before applying changes to ensure everything is correct.

### 4. **Show Execution Plan**
   ```
   terraform plan
   ```
   - **What it does:** Shows what changes Terraform will make without applying them.
   - **When to use:** Before running `apply` to review changes.

   **Using a `values.tfvars` file:**
   ```
   terraform plan -var-file="values.tfvars"
   ```
   - Use this when you have variables defined in `values.tfvars`.

### 5. **Apply Changes**
   ```
   terraform apply
   ```
   - **What it does:** Executes the planned changes and provisions resources.
   - **When to use:** After reviewing the `plan` output.

   **Using a `values.tfvars` file:**
   ```
   terraform apply -var-file="values.tfvars"
   ```

### 6. **Destroy Resources**
   ```
   terraform destroy
   ```
   - **What it does:** Destroys all resources managed by Terraform.
   - **When to use:** When decommissioning infrastructure.

   **Using a `values.tfvars` file:**
   ```
   terraform destroy -var-file="values.tfvars"
   ```

### 7. **Show State Information**
   ```
   terraform show
   ```
   - **What it does:** Displays the current state of Terraform-managed infrastructure.
   - **When to use:** To verify what Terraform has applied.

### 8. **List Resources in the State File**
   ```
   terraform state list
   ```
   - **What it does:** Lists all resources managed by Terraform.
   - **When to use:** When debugging or inspecting managed resources.

### 9. **Manually Update or Remove Resources from State**
   - **Remove a resource:**
     ```
     terraform state rm <resource_name>
     ```
   - **Move a resource to a different state:**
     ```
     terraform state mv <source> <destination>
     ```
   - **When to use:** When manually managing Terraform state.

### 10. **Output Terraform Values**
   ```
   terraform output
   ```
   - **What it does:** Displays output variables defined in `output.tf`.
   - **When to use:** To retrieve important values from your infrastructure.

---

## **Using `values.tfvars` for Variable Input**
- A **`values.tfvars`** file defines Terraform variables in a key-value format:

  ```hcl
  instance_type = "t2.micro"
  region = "us-east-1"
  ```

- To apply the values in this file, use:
  ```
  terraform plan -var-file="values.tfvars"
  terraform apply -var-file="values.tfvars"
  terraform destroy -var-file="values.tfvars"
  ```


# Old README file for Infrastructure via Terraformer
AWS Infrastructure via Terraform

**The Boritrader AWS Documentation**  
**Boritrade, LLC**  
**Updated Dec 30, 2024**  
**Maintainer:** Boritrade DevOps


## Overview
This document outlines the process of generating Terraform configuration files for Boritrade, LLC's AWS infrastructure using **Terraformer**. The Terraformer tool was used to import existing AWS resources into Terraform configuration files to enable infrastructure-as-code management.


## Setup

### **Prerequisites**
1. **Terraformer Installed**:
   - Ensure Terraformer is installed and up-to-date.
     [Terraformer Installation Guide](https://github.com/GoogleCloudPlatform/terraformer?tab=readme-ov-file#installation)

2. **AWS CLI Configured**:
   - Set up the AWS CLI with appropriate credentials and region.
     ```bash
     aws configure
     ```

3. **IAM Permissions**:
   - The IAM user or role used must have sufficient permissions to describe and manage AWS resources (e.g., `DescribeInstances`, `DescribeDBInstances`, etc.).

---

### **Import Commands**
The following commands were executed to generate the Terraform configuration files for each AWS service:

```bash
terraformer import aws --resources=route53 --regions=us-east-2
terraformer import aws --resources=alb --regions=us-east-2
terraformer import aws --resources=secretsmanager --regions=us-east-2
terraformer import aws --resources=ssm --regions=us-east-2
terraformer import aws --resources=rds --regions=us-east-2
terraformer import aws --resources=ec2_instance --regions=us-east-2
terraformer import aws --resources=wafv2_regional --regions=us-east-2
terraformer import aws --resources=vpc --regions=us-east-2
terraformer import aws --resources=sg --regions=us-east-2
terraformer import aws --resources=nacl --regions=us-east-2
terraformer import aws --resources=eip --regions=us-east-2
```

---

### **Resources Imported**
- **Route 53**: Imported hosted zones and DNS records.
- **Application Load Balancer (ALB)**: Captured load balancer configurations, including listeners and target groups.
- **Secrets Manager**: Imported secrets metadata, excluding sensitive values.
- **SSM Parameters**: Captured Parameter Store entries for managing application configurations.
- **RDS Databases**: Imported database instances, snapshots, and subnet groups.
- **EC2 Instances**: Captured running instance configurations, including AMI and instance type.
- **WAFv2 (Regional)**: Imported WAF WebACL and its association with the ALB.
- **VPC**: Captured networking infrastructure, including subnets, route tables, and gateways.
- **Security Groups**: Imported security groups and their associated rules.
- **Network ACLs**: Imported network ACLs and rules for subnet traffic control.
- **Elastic IPs**: Captured allocated EIPs for public-facing resources.


## Error Troubleshooting

### Common Errors and Fixes
#### Error: Missing Provider Plugins
```bash
└─$ terraformer import aws --resources=all --regions=us-east-2 --profile=default
2024/12/30 14:53:42 aws importing region us-east-2
2024/12/30 14:53:42 open /home/USER/.terraform.d/plugins/linux_amd64: no such file or directory
```
**Solution**:
1. In the folder where you intend to run Terraform, create a `main.tf` file:
   ```hcl
   terraform {
     required_providers {
       aws = {
         source  = "hashicorp/aws"
         version = "~> 4.16"
       }
     }

     required_version = ">= 1.2.0"
   }

   provider "aws" {
     region  = "us-west-2"
   }
   ```

2. Run the following command to install necessary provider plugins:
   ```bash
   terraform init
   ```

#### Error: Unsupported Service
If a specific resource type (e.g., `network_acl` or `security_group`) is not supported, ensure you are using the correct Terraformer resource name:
- **Security Groups**: `sg`
- **Network ACLs**: `nacl`


## Notes and Best Practices

### **State Management**
To ensure consistency and collaboration, use a remote backend for Terraform state:
```hcl
terraform {
  backend "s3" {
    bucket         = "your-terraform-state-bucket"
    key            = "path/to/terraform.tfstate"
    region         = "us-east-2"
    dynamodb_table = "terraform-locks"
  }
}
```

### **Resource Validation**
1. After running `terraform import`, always validate the configuration:
   ```bash
   terraform validate
   terraform plan
   ```
2. Verify all resource configurations (e.g., ensure proper tagging, IAM roles, and security groups).

### **Update WAF Association**
Ensure the `aws_wafv2_web_acl_association` resource is configured correctly:
```hcl
resource "aws_wafv2_web_acl_association" "example" {
  resource_arn = "arn:aws:elasticloadbalancing:us-east-2:USER:loadbalancer/app/NAME-Of-Balancer/ARN"
  web_acl_arn  = aws_wafv2_web_acl.tfer--CreatedByALB-Name-Of-Balancer.arn
}
```

### **Secrets and Sensitive Data**
Sensitive data (e.g., SSM & Secrets Manager values) should not be stored directly in `.tf` files. Use environment variables or external secure storage for handling secrets securely.


### Never destroy the following resources:
1. Route 52 (especially Hosted Zones)
2. RDS database (any)
3. ECS images (any)
4. ALB (R52 dns target?)
