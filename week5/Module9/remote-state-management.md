# Remote State Management & Terraform Workspaces

## Overview

Remote state management centralizes Terraform state files in secure, shared backends enabling team collaboration. Terraform workspaces manage multiple isolated environments (dev, staging, prod) within a single configuration codebase.

---

## Table of Contents

1. [State Fundamentals](#state-fundamentals)
2. [Why Remote State](#why-remote-state)
3. [Remote Backends](#remote-backends)
4. [State Locking](#state-locking)
5. [Terraform Workspaces](#terraform-workspaces)
6. [Best Practices](#best-practices)
7. [Security](#security)

---

## State Fundamentals

### What is Terraform State?

**State file** is a JSON document mapping Terraform configuration to real-world infrastructure

**Location**: `.terraform/terraform.tfstate` (local, default)

**Contains**:
- Resource IDs (provider-specific identifiers)
- Resource attributes (IPs, DNS names, configurations)
- Resource metadata (dependencies, counts)
- Input variable values
- Outputs

### State File Purpose

✅ Maps config to actual resources  
✅ Tracks infrastructure state  
✅ Enables plan/apply diff calculations  
✅ Stores metadata for resource dependencies  
✅ Enables safe concurrent operations (with locking)  

### Example State Structure

```json
{
  "version": 4,
  "terraform_version": "1.5.0",
  "resources": [
    {
      "type": "aws_instance",
      "name": "web",
      "instances": [
        {
          "attributes": {
            "ami": "ami-0abcdef...",
            "id": "i-0123456789abcdef0",
            "private_ip": "10.0.1.50",
            "public_ip": "203.0.113.45"
          }
        }
      ]
    }
  ]
}
```

---

## Why Remote State

### Problems with Local State

| Issue | Impact |
|-------|--------|
| **Collaboration conflicts** | Multiple devs overwrite each other's changes |
| **Security risks** | Credentials/IDs in version control |
| **No auditability** | Can't track who changed what |
| **Single point of failure** | Local machine crash = lost state |
| **No locking** | Concurrent applies corrupt state |

### Benefits of Remote State

✅ **Collaboration** - Multiple team members safely work together  
✅ **Centralized** - Single source of truth  
✅ **Secure** - Encryption, IAM, audit logs  
✅ **State locking** - Prevents concurrent modifications  
✅ **Version history** - Track state changes  
✅ **Disaster recovery** - Backed up automatically  
✅ **Auditing** - Complete change history  

### When to Use Remote State

✅ Team environments (always!)  
✅ Production/critical infrastructure  
✅ Multi-environment deployments  
✅ Audit/compliance requirements  
✅ Any project > 1 person  

---

## Remote Backends

### Backend Configuration

```hcl
terraform {
  backend "backend_type" {
    # Backend-specific arguments
  }
}
```

### 1. AWS S3 Backend

**Best for**: AWS-centric teams

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true                                 # ✅ Encrypt at rest
    dynamodb_table = "terraform-locks"                    # ✅ State locking
  }
}
```

**Setup Steps**:

```bash
# 1. Create S3 bucket
aws s3 mb s3://my-terraform-state --region us-east-1

# 2. Enable versioning
aws s3api put-bucket-versioning \
  --bucket my-terraform-state \
  --versioning-configuration Status=Enabled

# 3. Create DynamoDB table for locking
aws dynamodb create-table \
  --table-name terraform-locks \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5 \
  --region us-east-1

# 4. Initialize Terraform
terraform init
```

**Parameters**:
- `bucket`: S3 bucket name
- `key`: Path to state file (e.g., `env/prod/terraform.tfstate`)
- `region`: AWS region
- `encrypt`: Enable server-side encryption (recommended)
- `dynamodb_table`: DynamoDB table for locking (recommended)

### 2. Azure Blob Storage Backend

**Best for**: Azure-centric teams

```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "terraform-state-rg"
    storage_account_name = "tfstatestorage"
    container_name       = "tfstate"
    key                  = "prod/terraform.tfstate"
  }
}
```

**Setup Steps**:

```bash
# Create resource group
az group create --name terraform-state-rg --location eastus

# Create storage account
az storage account create \
  --name tfstatestorage \
  --resource-group terraform-state-rg \
  --location eastus

# Create blob container
az storage container create \
  --name tfstate \
  --account-name tfstatestorage

# Initialize Terraform
terraform init
```

**Parameters**:
- `resource_group_name`: Azure resource group
- `storage_account_name`: Storage account name
- `container_name`: Blob container name
- `key`: Path to state file

### 3. Terraform Cloud Backend

**Best for**: Enterprise teams, full-featured platform

```hcl
terraform {
  cloud {
    organization = "my-org"
    
    workspaces {
      name = "my-workspace"
    }
  }
}
```

**Setup Steps**:

```bash
# 1. Create Terraform Cloud account
# https://app.terraform.io/

# 2. Create organization and workspace in UI

# 3. Generate API token
# Account settings → Tokens

# 4. Configure authentication
export TF_TOKEN_app_terraform_io="your-api-token"

# 5. Initialize
terraform init
```

**Features**:
- Integrated state management
- Team/RBAC management
- VCS integration
- Built-in runs
- Policy as Code (Sentinel)
- Cost estimation
- Module registry

### Backend Comparison

| Feature | S3 | Azure Blob | TF Cloud |
|---------|----|----|----------|
| **Locking** | DynamoDB | Built-in | Built-in |
| **Encryption** | SSE | Built-in | Built-in |
| **Versioning** | Built-in | Built-in | Built-in |
| **Team features** | IAM | RBAC | Full |
| **Cost** | Low | Low | Free/Paid |
| **Complexity** | Medium | Medium | Low |

---

## State Locking

### What is State Locking?

**Lock mechanism** prevents concurrent modifications to state file

**Mechanism**: When apply/destroy starts, lock acquired. When done, lock released.

**Purpose**: Prevents corruption from simultaneous operations

### How It Works

```
User A: terraform apply
  ↓ Acquires lock on state file
  ↓ Makes changes
  ↓ Releases lock

User B: terraform apply (waits...)
  ↓ Lock is held by User A
  ↓ Waits for lock release
  ↓ Lock acquired
  ↓ Makes changes
  ↓ Releases lock
```

### Backends with Locking

| Backend | Locking Mechanism |
|---------|-------------------|
| **S3** | DynamoDB table |
| **Azure Blob** | Blob leases |
| **Terraform Cloud** | Built-in |
| **Consul** | Built-in |
| **Local** | File locking (basic) |

### S3 + DynamoDB Locking

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"  # ✅ Enable locking
  }
}
```

**DynamoDB table requirements**:
- Primary key: `LockID` (String)
- Must exist before Terraform init

### Troubleshooting Locks

```bash
# View DynamoDB lock
aws dynamodb scan --table-name terraform-locks

# Force release lock (⚠️ Use carefully!)
aws dynamodb delete-item \
  --table-name terraform-locks \
  --key '{"LockID":{"S":"my-terraform-state/terraform.tfstate"}}'
```

---

## Terraform Workspaces

### What are Workspaces?

**Workspaces** manage multiple isolated states within same configuration

**Use case**: Deploy same infrastructure to dev, staging, prod

**Each workspace** has its own state file

### Workspace vs Directory

| Method | Pros | Cons |
|--------|------|------|
| **Directories** | Complete isolation | Code duplication |
| **Workspaces** | Code reuse | State interdependency |

### Default Workspace

Every Terraform project starts with `default` workspace

```bash
terraform workspace show  # Output: default
```

### Workspace Commands

```bash
# List workspaces
terraform workspace list

# Create workspace
terraform workspace new dev
terraform workspace new staging
terraform workspace new prod

# Switch workspace
terraform workspace select dev

# Show current workspace
terraform workspace show

# Delete workspace (except default)
terraform workspace delete staging
```

### Remote State & Workspaces

When using remote backend, workspace name appended to state key

```
Backend key: global/terraform.tfstate

Results in:
- Default:  global/terraform.tfstate
- dev:      global/terraform.tfstate.dev
- staging:  global/terraform.tfstate.staging
- prod:     global/terraform.tfstate.prod
```

### Using Workspaces with Variables

**Recommended pattern**: Use workspaces + `.tfvars` files

**Project structure**:
```
.
├── main.tf
├── variables.tf
├── dev.tfvars
├── staging.tfvars
└── prod.tfvars
```

**variables.tf**:
```hcl
variable "instance_type" {
  type    = string
  default = "t2.micro"
}

variable "environment" {
  type = string
}

variable "enable_monitoring" {
  type    = bool
  default = false
}
```

**dev.tfvars**:
```hcl
environment        = "development"
instance_type      = "t2.micro"
enable_monitoring  = false
```

**staging.tfvars**:
```hcl
environment        = "staging"
instance_type      = "t2.small"
enable_monitoring  = true
```

**prod.tfvars**:
```hcl
environment        = "production"
instance_type      = "t3.large"
enable_monitoring  = true
```

**Workflow**:

```bash
# Development
terraform workspace select dev
terraform apply -var-file="dev.tfvars"

# Staging
terraform workspace select staging
terraform apply -var-file="staging.tfvars"

# Production
terraform workspace select prod
terraform apply -var-file="prod.tfvars"

# Switch between environments
terraform workspace select dev
terraform output instance_id  # Shows dev instance
```

### Workspace Reference in Code

Reference current workspace in code:

```hcl
# Access workspace name
terraform.workspace  # "dev", "staging", "prod", etc.

# Conditional logic
resource "aws_instance" "web" {
  instance_type = terraform.workspace == "prod" ? "t3.large" : "t2.micro"
  
  tags = {
    Environment = terraform.workspace
  }
}

# Alternative: Use variables (recommended)
variable "instance_type" {
  type = string
}

resource "aws_instance" "web" {
  instance_type = var.instance_type
  
  tags = {
    Environment = var.environment
  }
}
```

### Important Workspace Notes

✅ Run `terraform init` once per project  
✅ Switch workspaces with `terraform workspace select`  
✅ Each workspace has independent state  
✅ Use `.tfvars` files for environment-specific values  
✅ Cannot delete `default` workspace  
✅ Remote backend automatically isolates workspace states  

---

## Best Practices

### State Management

✅ **Always use remote state** for teams/production  
✅ **Enable state locking** on remote backends  
✅ **Encrypt state at rest** (S3 SSE, Azure encryption)  
✅ **Enable state versioning** for rollback capability  
✅ **Never commit state to Git** - add to `.gitignore`  
✅ **Backup state files** regularly  
✅ **Restrict state access** with IAM/RBAC  

### Workspace Management

✅ **Use descriptive workspace names** (dev, staging, prod)  
✅ **Use .tfvars files** for environment config  
✅ **Don't hardcode environment values** in HCL  
✅ **Document workspace purposes** in README  
✅ **Automate workspace creation** in CI/CD  
✅ **Clean up old workspaces** periodically  

### Team Collaboration

✅ **Document backend configuration** in README  
✅ **Require state lock review** before force unlock  
✅ **Use VCS for Terraform code** (Git)  
✅ **Code review all changes** via PRs  
✅ **Plan before apply** in CI/CD  
✅ **Archive old state** for audit  

---

## Security

### Access Control

```hcl
# Example: AWS S3 bucket policy
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::ACCOUNT:role/TerraformRole"
      },
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::my-terraform-state",
        "arn:aws:s3:::my-terraform-state/*"
      ]
    }
  ]
}
```

### Encryption

**S3 Backend**:
```hcl
terraform {
  backend "s3" {
    encrypt = true  # ✅ Server-side encryption
  }
}
```

**HTTPS**: Terraform always uses HTTPS for remote backends

### Secrets Management

❌ **DON'T**: Store secrets in state
```hcl
# BAD: Credentials in state
resource "aws_db_instance" "main" {
  password = "super-secret-password"  # ❌ Exposed in state!
}
```

✅ **DO**: Use external secret management
```hcl
# GOOD: Fetch from secrets manager
data "aws_secretsmanager_secret_version" "db_password" {
  secret_id = "prod/db/password"
}

resource "aws_db_instance" "main" {
  password = data.aws_secretsmanager_secret_version.db_password.secret_string
}
```

### Audit Logging

**S3**: Enable access logging
```bash
aws s3api put-bucket-logging \
  --bucket my-terraform-state \
  --bucket-logging-status '{
    "LoggingEnabled": {
      "TargetBucket": "my-audit-logs",
      "TargetPrefix": "tf-state-logs/"
    }
  }'
```

**Terraform Cloud**: Audit logs in UI

**Azure**: Storage analytics/monitoring

### Versioning & Rollback

**S3 Versioning**:
```bash
# List versions
aws s3api list-object-versions --bucket my-terraform-state

# Restore old version
aws s3api get-object \
  --bucket my-terraform-state \
  --key terraform.tfstate \
  --version-id VERSION_ID \
  terraform.tfstate.backup
```

---

## Quick Reference

### Initialization

```bash
# Configure backend
cat > backend.tf << 'EOF'
terraform {
  backend "s3" {
    bucket         = "my-state"
    key            = "terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}
EOF

# Initialize
terraform init
```

### Workspaces

```bash
terraform workspace list
terraform workspace new prod
terraform workspace select prod
terraform apply -var-file="prod.tfvars"
```

### State Commands

```bash
terraform state list
terraform state show aws_instance.web
terraform state rm aws_instance.old
terraform refresh
```

### Troubleshooting

```bash
# Show current state
terraform show

# Validate state integrity
terraform validate

# Debug state operations
TF_LOG=DEBUG terraform apply

# Check locks
aws dynamodb scan --table-name terraform-locks
```

---

## Common Issues & Solutions

| Issue | Solution |
|-------|----------|
| **Lock timeout** | Verify DynamoDB capacity, increase timeout |
| **State corruption** | Use versioning, restore from backup |
| **Lost local state** | Recover from remote backend with `terraform init` |
| **Backend migration fails** | Clear local cache, re-run `terraform init` |
| **Workspace already exists** | Delete with `terraform workspace delete` or use `select` |

---

## Summary

**Remote state**:
- Centralized, secure infrastructure state
- Enables team collaboration
- Built-in locking prevents corruption
- Available from S3, Azure, Terraform Cloud

**Workspaces**:
- Manage dev/staging/prod in single config
- Separate state per environment
- Combine with `.tfvars` for flexibility
- Perfect for multi-environment deployments

**Security**:
- Encrypt at rest & in transit
- Implement RBAC/IAM
- Use external secret management
- Enable audit logging
- Version state for rollback

---
