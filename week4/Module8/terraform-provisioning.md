# Terraform Commands & State Management - Complete Guide

## Overview
This guide covers the core Terraform commands (`init`, `plan`, `apply`, `destroy`) and essential state management concepts critical for managing infrastructure as code effectively.

---

## Part 1: Infrastructure as Code (IaC) Fundamentals

### What is Infrastructure as Code?

Infrastructure as Code is the practice of managing infrastructure through machine-readable definition files rather than manual configuration or interactive tools.

**Key Principles:**
- **Declarative Approach**: Define desired end state; the IaC tool determines how to achieve it
- **Automation**: Automated provisioning reduces manual effort and human error
- **Version Control**: Infrastructure definitions stored in Git enable tracking, collaboration, and rollback
- **Repeatability & Consistency**: Deploy identical infrastructure across environments (dev, staging, prod)
- **Modularity & Reusability**: Create reusable modules and components for efficiency

### Why IaC Matters

| Benefit | Impact |
|---------|--------|
| **Speed & Agility** | Deploy new environments in minutes, not weeks |
| **Reduced Risk** | Minimize human error through automation |
| **Cost Savings** | Efficient resource management and reduced manual labor |
| **Better Collaboration** | Teams can review, test, and debate infrastructure like code |
| **Enhanced Security** | Enforce policies and compliance standards directly in code |
| **Disaster Recovery** | Quickly recreate infrastructure in failover scenarios |

### Real-World Applications

- Cloud migration automation
- Environment management (dev/test/staging/prod)
- CI/CD pipeline integration
- Multi-cloud strategies
- On-premises automation
- Disaster recovery planning

---

## Part 2: Core Terraform Commands

### Command Workflow Overview

```
terraform init  →  terraform plan  →  terraform apply  →  terraform destroy
   Setup          Preview Changes      Provision          Cleanup
```

---

## `terraform init` - Preparing Your Workspace

### What Does It Do?

Initializes your Terraform project by:
- Scanning configuration files for required providers and modules
- Downloading provider plugins from the registry
- Initializing the backend for state storage
- Downloading referenced modules
- Creating `.terraform/` directory with dependencies

### Why It's Critical

- **Dependency Management**: Ensures all necessary tools are available
- **Environment Setup**: Configures state storage backend
- **Version Pinning**: Respects version constraints for reproducible deployments
- **Readiness**: Prepares directory for `plan` and `apply` commands

### Basic Usage

```bash
# Initial setup
terraform init

# Re-initialize with new providers/modules
terraform init

# Upgrade providers to latest compatible versions
terraform init -upgrade

# Reconfigure backend
terraform init -reconfigure

# Migrate state to new backend
terraform init -migrate-state
```

### Example: Initial Configuration

**main.tf:**
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  tags = {
    Name = "HelloWorldInstance"
  }
}
```

**Expected Output:**
```
Initializing the backend...
Initializing provider plugins...
- Finding hashicorp/aws versions matching "~> 5.0"...
- Installing hashicorp/aws v5.40.0...
- Installed hashicorp/aws v5.40.0 (signed by HashiCorp)

Terraform has been successfully initialized!
```

### Files Created After `init`

| File/Directory | Purpose |
|---|---|
| `.terraform/` | Contains provider plugins and modules |
| `.terraform.lock.hcl` | Records exact versions for reproducible builds |
| `.terraform/providers/` | Downloaded provider binaries |

---

## `terraform plan` - Previewing Infrastructure Changes

### What Does It Do?

Generates a dry-run execution plan showing:
- Resources to be **created** (+)
- Resources to be **modified** (~)
- Resources to be **destroyed** (-)
- Specific attribute changes

### Why It's Important

| Benefit | Description |
|---------|-------------|
| **Risk Mitigation** | Review before making changes; prevent accidental deletions |
| **Validation** | Catch syntax errors and logical inconsistencies early |
| **Impact Understanding** | See scope of changes before execution |
| **Cost Estimation** | Estimate resource costs based on planned changes |
| **Auditing** | Document intended changes for compliance |
| **Team Collaboration** | Share plans with stakeholders for review |

### Basic Usage

```bash
# Standard plan
terraform plan

# Save plan to file
terraform plan -out=tfplan

# View specific resource
terraform plan -var="instance_type=t3.medium"

# Destroy plan
terraform plan -destroy
```

### Example Output

```
Terraform used the selected backend "local" to store state.

Terraform will perform the following actions:

  # aws_instance.example will be created
  + resource "aws_instance" "example" {
      + ami                   = "ami-0c55b159cbfafe1f0"
      + instance_type         = "t2.micro"
      + id                    = (known after apply)
      + public_ip             = (known after apply)
      + tags                  = {
          + "Name" = "HelloWorldInstance"
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.
```

### Output Symbols Explained

| Symbol | Meaning |
|--------|---------|
| `+` | Resource will be created |
| `~` | Resource will be modified |
| `-` | Resource will be destroyed |
| `±` | Attribute may change |
| `(known after apply)` | Value determined after resource creation |

### Best Practices

✅ Always run `terraform plan` before `terraform apply`  
✅ Save plans to files: `terraform plan -out=tfplan`  
✅ Review plans with team members  
✅ Commit plans for audit trails  

---

## `terraform apply` - Provisioning Infrastructure

### What Does It Do?

Takes the execution plan and applies it to actual infrastructure by:
- Running an implicit `terraform plan` (unless plan file is provided)
- Prompting for confirmation
- Creating, modifying, or destroying resources via provider APIs
- Updating the state file with new infrastructure state
- Outputting resource attributes

### Why It's Important

- **Actual Resource Management**: Interacts with cloud providers to make real changes
- **State Updates**: Maintains accurate record of deployed infrastructure
- **Automation**: Eliminates manual infrastructure provisioning
- **Desired State Enforcement**: Ensures infrastructure matches configuration

### Basic Usage

```bash
# Apply with interactive confirmation
terraform apply

# Apply with saved plan (ensures reviewed plan is executed)
terraform apply tfplan

# Apply without confirmation (use cautiously!)
terraform apply -auto-approve

# Apply with variable overrides
terraform apply -var="instance_type=t3.medium"

# Apply with variable file
terraform apply -var-file="prod.tfvars"
```

### Complete Workflow Example

```bash
# Step 1: Review the plan
terraform plan -out=myplan.tfplan

# Step 2: Apply the saved plan (safest approach)
terraform apply myplan.tfplan
```

### Example Provisioning Output

```
aws_instance.example: Creating...
aws_instance.example: Still creating... [10s]
aws_instance.example: Still creating... [20s]
aws_instance.example: Creation complete after 25s [id=i-0123456789abcdef0]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

Outputs:

instance_public_ip = "54.123.45.67"
```

### Confirmation Prompt

```
Do you want to perform these actions?
  Terraform will execute this plan. Only 'yes' will be accepted to confirm.

  Enter a value: yes
```

### ⚠️ Important: -auto-approve Flag

```bash
terraform apply -auto-approve
```

**WARNING**: This bypasses the human review step. Use only in:
- Automated CI/CD pipelines with proper approval gates
- Non-production environments
- Trusted, tested configurations

---

## `terraform destroy` - Cleaning Up Resources

### What Does It Do?

Removes all infrastructure managed by Terraform by:
- Generating a destruction plan
- Prompting for confirmation
- Deleting resources in correct dependency order
- Updating state file to reflect destroyed resources

### Why It's Important

| Reason | Benefit |
|--------|---------|
| **Cost Management** | Eliminate unused resources to prevent unnecessary charges |
| **Environment Cleanup** | Remove temporary resources after testing/development |
| **Testing & Validation** | Test complete IaC workflow including destruction |
| **Disaster Recovery** | Practice full recreation of infrastructure |
| **Resource Control** | Prevent orphaned resources and infrastructure clutter |

### Basic Usage

```bash
# Destroy with confirmation prompt
terraform destroy

# Destroy without confirmation (use cautiously!)
terraform destroy -auto-approve

# Destroy with variable file
terraform destroy -var-file="prod.tfvars"

# Save destruction plan and apply
terraform plan -destroy -out=destroy.tfplan
terraform apply destroy.tfplan
```

### Example Destruction Output

```
Terraform will perform the following actions:

  # aws_instance.example will be destroyed
  - resource "aws_instance" "example" {
      - ami           = "ami-0c55b159cbfafe1f0" -> null
      - instance_type = "t2.micro" -> null
      - id            = "i-0123456789abcdef0" -> null
    }

Plan: 0 to add, 0 to change, 1 to destroy.

Do you really want to destroy all resources?
  Only 'yes' will be accepted to confirm.

  Enter a value: yes
```

### Destruction Progress

```
aws_instance.example: Destroying... [id=i-0123456789abcdef0]
aws_instance.example: Still destroying... [10s]
aws_instance.example: Still destroying... [20s]
aws_instance.example: Destruction complete after 25s

Destroy complete! Resources: 0 added, 0 changed, 1 destroyed.
```

### ⚠️ Critical Warnings

- **No Undo**: Destruction is permanent; deleted resources cannot be recovered
- **Data Loss**: Ensures all data is backed up before destroying databases or storage
- **Dependencies**: Some resources may fail to destroy if dependent resources exist
- **Confirmation Required**: Always confirm destruction carefully

---

## Part 3: Terraform State Files

### What is a State File?

A **state file** (`.tfstate`) is a JSON file that records Terraform's understanding of provisioned infrastructure. It maps resources in configuration to actual cloud resources.

### State File Contents

```json
{
  "version": 4,
  "terraform_version": "1.7.0",
  "serial": 1,
  "lineage": "unique-id",
  "outputs": {},
  "resources": [
    {
      "mode": "managed",
      "type": "aws_instance",
      "name": "example",
      "instances": [
        {
          "schema_version": 1,
          "attributes": {
            "ami": "ami-0c55b159cbfafe1f0",
            "instance_type": "t2.micro",
            "id": "i-0123456789abcdef0",
            "public_ip": "54.123.45.67",
            "tags": {
              "Name": "HelloWorldInstance"
            }
          }
        }
      ]
    }
  ]
}
```

### What State Tracks

- **Resource Type**: `aws_instance`, `aws_s3_bucket`, etc.
- **Resource ID**: Unique identifier from cloud provider
- **Resource Attributes**: Current values (IP addresses, ARNs, status)
- **Dependencies**: Resource relationships
- **Module Information**: Module structure and mappings

### Why State Files Are Critical

| Aspect | Importance |
|--------|-----------|
| **Resource Tracking** | Terraform knows which resources it manages |
| **Drift Detection** | Identifies infrastructure changed outside Terraform |
| **Dependencies** | Determines correct order for creation/destruction |
| **Attribute Values** | References for other resources (e.g., subnet ID) |
| **Updates & Deletions** | Without state, Terraform can't identify what to change |

### Local State File Risks

⚠️ **Problems with local state in team/production environments:**

- **Concurrency Issues**: Multiple users overwrite each other's state
- **State Loss**: Accidental deletion causes loss of infrastructure memory
- **Lack of Access Control**: No granular permissions on state file
- **No Audit Trail**: Difficult to track who changed what
- **Backup Challenges**: Manual backup creates inconsistencies

### State File Best Practices

```hcl
# ✅ DO NOT modify state files manually
terraform state list              # View managed resources
terraform state show aws_instance.example  # Show resource details
terraform state rm aws_instance.example    # Remove resource from state

# ✅ Use version control for code, NOT state files
# Add to .gitignore:
# terraform.tfstate
# terraform.tfstate.*
# .terraform/
```

---

## Part 4: Remote State Management

### What is Remote State?

Remote state means storing `.tfstate` in a centralized, shared location instead of locally. Team members and automation systems access the same state.

### Supported Remote Backends

| Backend | Use Case |
|---------|----------|
| **S3 + DynamoDB** | AWS-based, state locking, versioning |
| **Azure Blob Storage** | Microsoft Azure environments |
| **Google Cloud Storage** | Google Cloud Platform |
| **Terraform Cloud/Enterprise** | HashiCorp managed, full-featured |
| **Consul** | HashiCorp Consul backend |

### Benefits of Remote State

| Benefit | Description |
|---------|-------------|
| **Collaboration** | Multiple team members work simultaneously |
| **State Locking** | Prevents concurrent modifications; only one user at a time |
| **Security** | Encryption, access controls, audit logs |
| **Durability** | Cloud storage provides high availability and backups |
| **CI/CD Integration** | Automation servers safely access shared state |
| **Centralized Management** | Single source of truth for infrastructure |

### How State Locking Works

```
User 1: terraform apply
  ↓
Lock acquired on state file
  ↓
User 1: Makes changes
  ↓
State updated and uploaded
  ↓
Lock released
  ↓
User 2: Now can run terraform apply
```

---

## Setting Up Remote State with AWS S3

### Step 1: Create S3 Bucket

```bash
# Via AWS CLI
aws s3api create-bucket \
  --bucket my-terraform-state-bucket-unique \
  --region us-east-1

# Enable versioning
aws s3api put-bucket-versioning \
  --bucket my-terraform-state-bucket-unique \
  --versioning-configuration Status=Enabled

# Block public access (security)
aws s3api put-public-access-block \
  --bucket my-terraform-state-bucket-unique \
  --public-access-block-configuration \
  "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"
```

### Step 2: Create DynamoDB Table for State Locking

```bash
aws dynamodb create-table \
  --table-name my-terraform-state-lock \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5 \
  --region us-east-1
```

### Step 3: Configure Backend in Terraform

**backend.tf** or **main.tf:**
```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state-bucket-unique"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "my-terraform-state-lock"
    encrypt        = true  # Enable encryption
  }
}

provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  tags = {
    Name = "HelloWorldInstance"
  }
}
```

### Step 4: Re-initialize Terraform

```bash
terraform init
```

**Output:**
```
Initializing the backend...
Successfully configured the backend "s3"! Terraform will automatically
use this backend unless the backend configuration changes.

Terraform has been successfully initialized!
```

### Step 5: Verify Remote State

```bash
# State is now stored in S3
aws s3 ls s3://my-terraform-state-bucket-unique/
  PRE prod/

aws s3 ls s3://my-terraform-state-bucket-unique/prod/
  2024-01-15 10:30:00     12345 terraform.tfstate
```

### State Operations with Remote Backend

```bash
# Download and display state
terraform state list

# Show specific resource
terraform state show aws_instance.example

# Apply automatically uses remote state
terraform apply  # Downloads state, locks, applies, uploads, unlocks

# Force unlock (use only in emergencies)
terraform force-unlock LOCK_ID
```

---

## Remote State Best Practices

✅ **Enable Versioning**: Recover from accidental corruption  
✅ **Encrypt at Rest**: Use S3 encryption, Azure encryption, etc.  
✅ **Enable State Locking**: Prevent concurrent modifications  
✅ **Restrict Access**: IAM policies limit who can access state  
✅ **Separate State per Environment**: Different buckets/keys for dev/staging/prod  
✅ **Enable MFA Delete**: Require MFA for state bucket deletion  
✅ **Audit Logging**: CloudTrail logs all state access  
✅ **Regular Backups**: Though S3 versioning provides this  
✅ **Consider Terraform Cloud**: Full-featured remote state management  

---

## Part 5: Complete Hands-On Workflow

### Project Structure

```
terraform-ec2-demo/
├── main.tf           # Provider and resources
├── variables.tf      # Input variables
├── outputs.tf        # Output values
├── backend.tf        # Remote state configuration
├── dev.tfvars        # Development environment variables
├── prod.tfvars       # Production environment variables
├── .terraform/       # Provider plugins (auto-created)
├── .terraform.lock.hcl # Provider versions (auto-created)
├── terraform.tfstate # Local state (if not using remote)
└── .gitignore        # Exclude sensitive files
```

### Step-by-Step: Complete Lifecycle

#### Step 1: Create Configuration

**main.tf:**
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }

  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "ec2/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-lock"
    encrypt        = true
  }
}

provider "aws" {
  region = var.aws_region
}

resource "aws_instance" "my_ec2_instance" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = var.instance_type

  tags = merge(var.common_tags, {
    Name = "MyTerraformInstance"
  })
}
```

**variables.tf:**
```hcl
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}

variable "common_tags" {
  description = "Common tags for all resources"
  type        = map(string)
  default = {
    Environment = "Dev"
    ManagedBy   = "Terraform"
  }
}
```

**outputs.tf:**
```hcl
output "instance_public_ip" {
  description = "Public IP of EC2 instance"
  value       = aws_instance.my_ec2_instance.public_ip
}

output "instance_id" {
  description = "EC2 instance ID"
  value       = aws_instance.my_ec2_instance.id
}
```

**dev.tfvars:**
```hcl
aws_region    = "us-east-1"
instance_type = "t2.micro"
common_tags = {
  Environment = "Development"
  ManagedBy   = "Terraform"
  Owner       = "DevTeam"
}
```

**.gitignore:**
```
# Local state files
terraform.tfstate
terraform.tfstate.*
*.tfstate.backup

# Terraform directories
.terraform/
.terraform.lock.hcl

# Variable files with secrets
*.tfvars
!*.tfvars.example

# IDE files
.vscode/
*.swp
*.swo
```

#### Step 2: Initialize

```bash
cd terraform-ec2-demo
terraform init

# Output:
# Initializing the backend...
# Successfully configured the backend "s3"!
# Terraform has been successfully initialized!
```

#### Step 3: Plan

```bash
terraform plan -var-file="dev.tfvars" -out=dev.tfplan

# Output shows:
# Terraform will perform the following actions:
#   # aws_instance.my_ec2_instance will be created
#   + resource "aws_instance" "my_ec2_instance" {
#       + ami           = "ami-0c55b159cbfafe1f0"
#       + instance_type = "t2.micro"
#       ...
#     }
# 
# Plan: 1 to add, 0 to change, 0 to destroy.
```

#### Step 4: Apply

```bash
terraform apply dev.tfplan

# Output:
# aws_instance.my_ec2_instance: Creating...
# aws_instance.my_ec2_instance: Creation complete after 25s [id=i-0abc123def456]
# 
# Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
# 
# Outputs:
# instance_public_ip = "54.123.45.67"
# instance_id = "i-0abc123def456"
```

#### Step 5: Verify

```bash
# View outputs
terraform output

# Check state
terraform state list
# aws_instance.my_ec2_instance

terraform state show aws_instance.my_ec2_instance

# Verify in AWS
aws ec2 describe-instances --region us-east-1 \
  --query 'Reservations[0].Instances[0].[InstanceId,State.Name,PublicIpAddress]'
```

#### Step 6: Destroy

```bash
terraform destroy -var-file="dev.tfvars"

# Confirmation prompt:
# Do you really want to destroy all resources?
# Only 'yes' will be accepted to confirm.
# 
# Enter a value: yes

# Output:
# aws_instance.my_ec2_instance: Destroying... [id=i-0abc123def456]
# aws_instance.my_ec2_instance: Destruction complete after 15s
# 
# Destroy complete! Resources: 0 added, 0 changed, 1 destroyed.
```

---

## Quick Reference: Common Commands

| Task | Command |
|------|---------|
| **Initialize project** | `terraform init` |
| **Validate syntax** | `terraform validate` |
| **Format code** | `terraform fmt` |
| **Preview changes** | `terraform plan -out=tfplan` |
| **Apply changes** | `terraform apply tfplan` |
| **View outputs** | `terraform output` |
| **List resources** | `terraform state list` |
| **Show resource details** | `terraform state show <resource>` |
| **Remove resource from state** | `terraform state rm <resource>` |
| **Destroy infrastructure** | `terraform destroy` |
| **Auto-approve (dangerous!)** | `terraform apply -auto-approve` |
| **Force unlock** | `terraform force-unlock <LOCK_ID>` |

---

## Troubleshooting Guide

### Common Issues & Solutions

| Issue | Cause | Solution |
|-------|-------|----------|
| **Plan shows unexpected changes** | Configuration mismatch or manual changes | Review configuration and run `terraform refresh` |
| **State lock timeout** | Another user is applying changes | Wait for lock to release or use `force-unlock` |
| **"Resource already exists" error** | Resource created outside Terraform | Import with `terraform import` or remove from state |
| **Authentication failures** | Invalid credentials | Verify AWS CLI config or environment variables |
| **AMI not found** | Invalid AMI ID for region | Check AWS EC2 console for correct AMI ID |
| **Destroy fails** | Dependent resources | Review error; manually delete dependent resources |

---

## Key Takeaways

✅ **IaC Benefits**: Speed, reliability, consistency, cost savings, collaboration  
✅ **terraform init**: Sets up dependencies and backend  
✅ **terraform plan**: Always review before applying changes  
✅ **terraform apply**: Provisions actual infrastructure  
✅ **terraform destroy**: Cleans up resources to manage costs  
✅ **State Files**: Critical for tracking managed resources  
✅ **Remote State**: Essential for teams and production  
✅ **State Locking**: Prevents concurrent modifications  
✅ **Best Practices**: Version control, remote state, encryption, access controls  

---

## Essential Best Practices Summary

### Development
- Use local backend with state in `.gitignore`
- Commit `.terraform.lock.hcl` for reproducibility
- Frequently destroy/recreate to test full workflow

### Staging/Production
- **ALWAYS use remote state** (S3, Azure, GCP)
- Enable state locking
- Enable encryption at rest
- Enable versioning for recovery
- Restrict IAM access strictly
- Enable CloudTrail logging
- Use separate state files per environment
- Require plan review before apply
- Use CI/CD with approval gates

### Team Collaboration
- Store code in Git (not state files)
- Use remote state for shared infrastructure
- Implement code review process for `.tf` changes
- Document variable requirements
- Use consistent naming conventions
- Share `.tfvars.example` (without secrets)
- Regular state audits

---
