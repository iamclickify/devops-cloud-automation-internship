# Advanced Terraform Concepts & Best Practices

## Overview

Advanced Terraform techniques enable robust, scalable, and maintainable infrastructure as code through provisioners, code quality tools, dependency locking, strategic project structuring, and automated testing.

---

## Table of Contents

1. [Terraform Provisioners](#terraform-provisioners)
2. [Code Quality Tools](#code-quality-tools)
3. [Dependency Locking](#dependency-locking)
4. [Project Architecture](#project-architecture)
5. [Testing with Terratest](#testing-with-terratest)
6. [Troubleshooting](#troubleshooting)
7. [Best Practices](#best-practices)

---

## Terraform Provisioners

### What are Provisioners?

**Provisioners** are meta-arguments that execute scripts/commands on local or remote machines during resource lifecycle

**Two types**:
- `local-exec`: Commands on Terraform machine
- `remote-exec`: Commands on remote resource (SSH/WinRM)

### When to Use Provisioners

✅ **Good use cases**:
- Software installation
- Service configuration
- Database schema setup
- Application initialization
- Cleanup before destroy

❌ **Avoid for**:
- Complex configuration (use Ansible/Chef instead)
- Long-running processes
- Frequent updates

### local-exec Provisioner

```hcl
resource "null_resource" "example" {
  provisioner "local-exec" {
    command = "echo 'Creating resource at $(date)'"
  }
}

# With environment variables
resource "null_resource" "build" {
  provisioner "local-exec" {
    command = "cd ${path.module} && ./build.sh"
    environment = {
      BUILD_ENV = "production"
    }
  }
}

# Run on destroy
resource "aws_instance" "web" {
  provisioner "local-exec" {
    when    = destroy
    command = "echo 'Destroying instance: ${self.id}'"
  }
}
```

### remote-exec Provisioner

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  key_name      = "my-key"

  vpc_security_group_ids = [aws_security_group.allow_ssh.id]

  # Inline commands
  provisioner "remote-exec" {
    inline = [
      "sudo apt-get update -y",
      "sudo apt-get install -y nginx",
      "sudo systemctl start nginx",
      "sudo systemctl enable nginx"
    ]

    connection {
      type        = "ssh"
      user        = "ubuntu"
      private_key = file("~/.ssh/my-key.pem")
      host        = self.public_ip
    }
  }
}

# Script file execution
resource "aws_instance" "app" {
  provisioner "remote-exec" {
    script = "${path.module}/setup.sh"

    connection {
      type        = "ssh"
      user        = "ec2-user"
      private_key = file("~/.ssh/my-key.pem")
      host        = self.public_ip
    }
  }
}
```

### Provisioner Parameters

| Parameter | Purpose | Values |
|-----------|---------|--------|
| **command** (local-exec) | Command to run | Shell command string |
| **inline** (remote-exec) | List of commands | Array of strings |
| **script** (remote-exec) | Script file path | Path to script |
| **when** | Lifecycle trigger | create, destroy |
| **on_failure** | Failure behavior | continue, fail |
| **connection** | Remote access | SSH/WinRM settings |

### Provisioner Best Practices

✅ **Ensure idempotency** - Run multiple times safely  
✅ **Handle errors gracefully** - Use `on_failure = continue` when appropriate  
✅ **Keep scripts simple** - Complex logic belongs in config management  
✅ **Use SSH key pairs** - Secure remote access  
✅ **Clean up on destroy** - Use destroy provisioners  
✅ **Log operations** - For debugging  

❌ **Avoid hardcoding secrets** - Use variables/vault  
❌ **Don't use for complex config** - Use Ansible, Chef, Puppet  
❌ **Avoid long-running commands** - Can timeout  

---

## Code Quality Tools

### terraform fmt - Code Formatting

**Purpose**: Automatically format Terraform code to canonical style

**Benefits**:
- Consistent formatting
- Reduced diff noise
- Team collaboration
- Automated enforcement

```bash
# Format current directory
terraform fmt

# Format recursively
terraform fmt -recursive

# Check format without modifying
terraform fmt -check

# Show what will change
terraform fmt -diff
```

**Integration**:
```bash
# Pre-commit hook
#!/bin/bash
terraform fmt -check -recursive .
if [ $? -ne 0 ]; then
  echo "Terraform files need formatting"
  exit 1
fi
```

### terraform validate - Configuration Validation

**Purpose**: Check syntax and internal consistency

**Validates**:
- Syntax correctness
- Resource type existence
- Variable declarations
- Module references
- Provider requirements

```bash
# Validate configuration
terraform validate

# Output on success
# Success! The configuration is valid.

# Output on error
# Error: Unsupported argument
# 
#   on main.tf line 5, in resource "aws_instance" "example":
#    5:   invalid_argument = "value"
```

**When to run**:
- After writing HCL
- In CI/CD pipelines
- Pre-commit hooks
- Before plan/apply

### CI/CD Integration

```yaml
# GitHub Actions example
name: Terraform Validate

on: [push, pull_request]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - uses: hashicorp/setup-terraform@v2
      
      - name: Terraform Format Check
        run: terraform fmt -check -recursive
      
      - name: Terraform Validate
        run: terraform validate
```

---

## Dependency Locking

### What is Dependency Locking?

**Purpose**: Pin exact versions of providers/modules for reproducible builds

**File**: `.terraform.lock.hcl` (auto-generated)

**Benefits**:
✅ Reproducible deployments  
✅ Prevent unexpected updates  
✅ Team consistency  
✅ Audit trail  
✅ Security (no surprise updates)  

### How It Works

```hcl
# versions.tf - Define version constraints
terraform {
  required_version = ">= 1.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = ">= 5.0, < 6.0"
    }
    random = {
      source  = "hashicorp/random"
      version = "~> 3.0"
    }
  }
}
```

**First `terraform init`**:
- Downloads latest versions matching constraints
- Creates `.terraform.lock.hcl` with exact versions
- Records cryptographic hashes

**Subsequent `terraform init`**:
- Reads lock file
- Downloads exact pinned versions
- Ignores constraint ranges

### Lock File Example

```hcl
# .terraform.lock.hcl

terraform {
  version = ">= 1.0"
}

provider "registry.terraform.io/hashicorp/aws" {
  version     = "5.30.0"
  constraints = ">= 5.0, < 6.0"
  hashes = [
    "h1:...",
    "zh:...",
  ]
}

provider "registry.terraform.io/hashicorp/random" {
  version     = "3.5.1"
  constraints = "~> 3.0"
  hashes = [
    "h1:...",
  ]
}
```

### Managing Lock File

```bash
# View lock file
cat .terraform.lock.hcl

# Update to new versions
terraform init -upgrade

# Force specific provider version
# Edit lock file or:
terraform init -upgrade -json | jq '.provider_constraints'

# Commit to Git
git add .terraform.lock.hcl
git commit -m "Lock Terraform provider versions"
```

**CRITICAL**: Always commit `.terraform.lock.hcl` to version control!

---

## Project Architecture

### Module-Based Structure

**Recommended structure**:
```
project/
├── modules/                    # Reusable modules
│   ├── vpc/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── security_groups/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   └── database/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
│
├── environments/               # Environment-specific configs
│   ├── dev/
│   │   ├── main.tf
│   │   ├── backend.tf
│   │   ├── variables.tf
│   │   └── terraform.tfvars
│   ├── staging/
│   │   ├── main.tf
│   │   ├── backend.tf
│   │   ├── variables.tf
│   │   └── terraform.tfvars
│   └── prod/
│       ├── main.tf
│       ├── backend.tf
│       ├── variables.tf
│       └── terraform.tfvars
│
├── README.md
└── .terraform.lock.hcl
```

### Module Design

**Principles**:
✅ Single responsibility  
✅ Focused scope  
✅ Clear inputs/outputs  
✅ Well documented  
✅ Versioned  

**Example module**:
```hcl
# modules/vpc/main.tf
resource "aws_vpc" "main" {
  cidr_block           = var.cidr_block
  enable_dns_hostnames = true

  tags = {
    Name        = "${var.environment}-vpc"
    Environment = var.environment
  }
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = {
    Name = "${var.environment}-igw"
  }
}

# modules/vpc/variables.tf
variable "cidr_block" {
  type = string
}

variable "environment" {
  type = string
}

# modules/vpc/outputs.tf
output "vpc_id" {
  value = aws_vpc.main.id
}

output "igw_id" {
  value = aws_internet_gateway.main.id
}
```

### Environment Configuration

```hcl
# environments/prod/main.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  
  backend "s3" {
    bucket         = "terraform-state-prod"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}

provider "aws" {
  region = var.aws_region
}

module "vpc" {
  source = "../../modules/vpc"
  
  cidr_block  = var.vpc_cidr
  environment = "prod"
}

module "security_groups" {
  source = "../../modules/security_groups"
  
  vpc_id      = module.vpc.vpc_id
  environment = "prod"
}

output "vpc_id" {
  value = module.vpc.vpc_id
}
```

```hcl
# environments/prod/terraform.tfvars
aws_region = "us-east-1"
vpc_cidr   = "10.0.0.0/16"
```

### Workspace Alternative

```bash
# Create workspaces
terraform workspace new dev
terraform workspace new staging
terraform workspace new prod

# Use in code
resource "aws_instance" "app" {
  instance_type = terraform.workspace == "prod" ? "t3.large" : "t2.micro"
  
  tags = {
    Environment = terraform.workspace
  }
}

# Apply per environment
terraform workspace select prod
terraform apply -var-file="prod.tfvars"
```

### Key Principles

✅ **Keep modules small** - Single responsibility  
✅ **Clear dependencies** - Use module outputs  
✅ **Document modules** - README for each  
✅ **Version modules** - Use Git tags  
✅ **Remote state** - S3 or Terraform Cloud  
✅ **Environment separation** - Distinct state per env  

---

## Testing with Terratest

### What is Terratest?

**Terratest** is a Go library for testing infrastructure code

**Capabilities**:
- Apply Terraform configurations
- Verify deployed resources
- Assert expected outcomes
- Clean up after tests

### Why Test Infrastructure?

✅ Confidence in deployments  
✅ Early bug detection  
✅ Safe refactoring  
✅ Compliance verification  
✅ Module validation  

### Terratest Example

```go
// main_test.go
package main

import (
	"testing"
	
	"github.com/gruntwork-io/terratest/modules/terraform"
	"github.com/stretchr/testify/assert"
)

func TestTerraformVPC(t *testing.T) {
	tfOpts := &terraform.Options{
		TerraformDir: "../examples/vpc",
	}
	
	// Cleanup after test
	defer terraform.Destroy(t, tfOpts)
	
	// Apply configuration
	terraform.InitAndApply(t, tfOpts)
	
	// Get outputs
	vpcID := terraform.Output(t, tfOpts, "vpc_id")
	
	// Assert
	assert.NotEmpty(t, vpcID)
	t.Logf("VPC ID: %s", vpcID)
}
```

### Setup

```bash
# Create Go project
mkdir terratest-example
cd terratest-example
go mod init example.com/terratest

# Install Terratest
go get github.com/gruntwork-io/terratest
go get github.com/stretchr/testify/assert

# Run tests
go test -v -timeout 30m
```

### Advanced Testing

```go
// Test with variables
tfOpts := &terraform.Options{
	TerraformDir: "../examples/vpc",
	Vars: map[string]interface{}{
		"vpc_cidr": "10.0.0.0/16",
	},
}

// Test multiple scenarios
for _, env := range []string{"dev", "staging", "prod"} {
	t.Run(env, func(t *testing.T) {
		opts := &terraform.Options{
			TerraformDir: fmt.Sprintf("../examples/%s", env),
		}
		defer terraform.Destroy(t, opts)
		terraform.InitAndApply(t, opts)
		// Verify env-specific behavior
	})
}
```

### CI/CD Integration

```yaml
# GitHub Actions
- name: Run Terratest
  run: |
    cd tests
    go test -v -timeout 30m ./...
  env:
    AWS_REGION: us-east-1
    AWS_ACCOUNT_ID: ${{ secrets.AWS_ACCOUNT_ID }}
```

---

## Troubleshooting

### State File Issues

| Issue | Solution |
|-------|----------|
| **Corruption** | Use backup, `terraform state rm/mv`, remote locking |
| **Drift** | `terraform refresh`, manual import with `terraform import` |
| **Lost state** | Restore from backup or re-import resources |

### Provider Issues

| Issue | Solution |
|-------|----------|
| **Version conflict** | Use `.terraform.lock.hcl`, pin versions |
| **Breaking changes** | Review release notes before upgrade |
| **Auth failure** | Verify credentials, IAM permissions |

### Resource Creation Failures

```bash
# Enable debug logging
export TF_LOG=DEBUG
terraform apply

# Check error messages
# Verify cloud permissions
# Check resource limits/quotas
# Review provider logs
```

### Cyclic Dependencies

```hcl
# ❌ Bad: Circular dependency
resource "aws_instance" "app" {
  security_groups = [aws_security_group.app.id]
}

resource "aws_security_group" "app" {
  cidr_blocks = [aws_instance.app.private_ip]  # Cycle!
}

# ✅ Good: Break cycle with data source
data "aws_instance" "app" {
  id = "i-1234567890abcdef0"
}

resource "aws_security_group" "app" {
  cidr_blocks = ["${data.aws_instance.app.private_ip}/32"]
}
```

### Performance Issues

```bash
# Large state files slow operations
# Solutions:
# 1. Split into multiple state files
# 2. Use data sources for read-only resources
# 3. Move to dedicated configurations
# 4. Upgrade to faster remote backend

# Diagnose with timing
terraform apply -input=false 2>&1 | grep -E "^Time|^Error"
```

### Common Commands

```bash
# Debug state
terraform state list
terraform state show aws_instance.web
terraform state rm aws_instance.old

# Refresh without changes
terraform refresh

# Validate and format
terraform validate
terraform fmt -recursive

# Detailed logging
TF_LOG=TRACE terraform apply

# Save plan for review
terraform plan -out=tfplan
terraform apply tfplan
```

---

## Best Practices

### Code Quality

✅ **Run `terraform fmt`** - In pre-commit or CI  
✅ **Run `terraform validate`** - Before apply  
✅ **Use meaningful names** - Resources, variables, outputs  
✅ **Comment complex logic** - Explain why, not what  
✅ **Keep DRY principle** - Use modules/locals  

### Security

✅ **Never hardcode secrets** - Use Vault/Secrets Manager  
✅ **Enable encryption** - State, backups, in-transit  
✅ **Restrict state access** - IAM/RBAC  
✅ **Enable audit logging** - Track state changes  
✅ **Use service accounts** - Minimal permissions  

### Collaboration

✅ **Commit lock file** - Consistent versions  
✅ **Use remote state** - Central source of truth  
✅ **Enable state locking** - Prevent conflicts  
✅ **Code review changes** - Plan review before apply  
✅ **Document architecture** - README/diagrams  

### Operations

✅ **Test infrastructure** - Terratest/manual  
✅ **Plan before apply** - Never skip  
✅ **Backup state** - Versioning enabled  
✅ **Monitor drift** - Regular refresh  
✅ **Version everything** - Providers, modules, tools  

### Design

✅ **Modular architecture** - Reusable components  
✅ **Clear separation** - Modules vs environments  
✅ **Single responsibility** - Focus each module  
✅ **Composition** - Modules work together  
✅ **Scalability** - Handles growth  

---

## Quick Reference

### Essential Commands

```bash
# Formatting
terraform fmt -recursive

# Validation
terraform validate

# State management
terraform state list
terraform state show [resource]
terraform state rm [resource]
terraform import [resource] [id]

# Logging
TF_LOG=DEBUG terraform apply
TF_LOG_PATH=tf.log terraform apply

# Testing
go test -v -timeout 30m ./...

# Lock file
terraform init -upgrade
```

### Project Layout Checklist

- [ ] `modules/` directory with focused modules
- [ ] `environments/` directory per environment
- [ ] `versions.tf` with provider constraints
- [ ] `backend.tf` with remote state config
- [ ] `.terraform.lock.hcl` committed to Git
- [ ] `.gitignore` excludes `.terraform/`
- [ ] README.md documents architecture
- [ ] Module READMEs document usage
- [ ] Test files for critical modules

---

## Summary

**Advanced Terraform** requires:

**Provisioners**: Automate lifecycle tasks (install software, config)

**Code Quality**: Use `fmt` and `validate` to catch issues early

**Dependency Locking**: Commit `.terraform.lock.hcl` for reproducibility

**Architecture**: Module-based design with environment separation

**Testing**: Terratest validates infrastructure behavior

**Troubleshooting**: Understand state, logging, error messages

**Best Practices**: Security, collaboration, testing, design

---

