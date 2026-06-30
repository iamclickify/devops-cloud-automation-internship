# Infrastructure as Code (IaC) & Terraform Fundamentals

## Overview

Infrastructure as Code is the practice of managing and provisioning IT infrastructure through machine-readable definition files rather than manual configuration. Terraform is the leading cloud-agnostic IaC tool enabling declarative infrastructure management across multiple cloud providers.

---

## Table of Contents

1. [What is IaC](#what-is-iac)
2. [Core Principles](#core-principles)
3. [Benefits of IaC](#benefits-of-iac)
4. [Declarative vs Imperative](#declarative-vs-imperative)
5. [Popular IaC Tools](#popular-iac-tools)
6. [Terraform Architecture](#terraform-architecture)
7. [Terraform Workflow](#terraform-workflow)
8. [State Management](#state-management)
9. [Best Practices](#best-practices)

---

## What is IaC

### Definition

**Infrastructure as Code** is the management and provisioning of IT infrastructure (servers, networks, databases, load balancers, storage) through machine-readable definition files instead of manual clicks or interactive configuration tools.

### Key Characteristics

✅ Infrastructure defined in code files  
✅ Version controlled (Git)  
✅ Automated provisioning and management  
✅ Repeatable and consistent  
✅ Reviewable and auditable  
✅ Treated like application code  

### Real-World Applications

| Use Case | Benefit |
|----------|---------|
| **Multi-environment setup** | Same config for dev, staging, prod |
| **Disaster recovery** | Quickly recreate entire infrastructure |
| **Scaling** | Provision additional resources on demand |
| **Team collaboration** | Review and approve infra changes via PR |
| **Cost optimization** | Automated cleanup of temporary resources |
| **Compliance** | Infrastructure audits and change tracking |

---

## Core Principles

### 1. Declarative Approach

**What you want** → Define desired end state → Tool figures out how

```hcl
resource "aws_s3_bucket" "my_bucket" {
  bucket = "my-bucket-name"
  tags = {
    Environment = "Production"
  }
}
```

**Benefits**: Simpler, idempotent, easy to understand desired state

### 2. Idempotency

Running same configuration multiple times = same final state (no side effects)

**Example**: 
- First run: Creates bucket
- Second run: No changes (already exists and matches config)
- Third run: No changes (still matches)

### 3. Version Control

Infrastructure definitions stored in Git
- Track changes (who, when, why)
- Revert to previous versions
- Code review process
- Team collaboration

### 4. Automation

Entire lifecycle automated
- Provisioning
- Configuration
- Updates
- Destruction

Reduces manual effort and human error

### 5. Modularity & Reusability

Break infrastructure into reusable modules
- Promote consistency
- Reduce duplication
- Enable sharing across projects

---

## Benefits of IaC

### Speed & Efficiency

| Benefit | Impact |
|---------|--------|
| **Faster provisioning** | Minutes instead of hours for setup |
| **Parallel resource creation** | Multiple resources created simultaneously |
| **Reduced setup time** | Automated environment creation |

### Consistency & Reliability

✅ Identical environments across dev/staging/prod  
✅ Eliminate configuration drift  
✅ No "it works on my machine" problems  
✅ Deterministic outcomes  

### Reduced Risk

✅ Minimize human error  
✅ Automated, tested processes  
✅ Peer review before deployment  
✅ Easy rollback capabilities  

### Cost Savings

✅ Optimize resource utilization  
✅ Automate resource cleanup  
✅ Prevent costly misconfigurations  
✅ Efficient resource allocation  

### Improved Collaboration

✅ Infrastructure in version control  
✅ Change review and approval process  
✅ Documentation built into code  
✅ Cross-team understanding  

### Enhanced Security

✅ Security policies embedded in code  
✅ Compliance from the start  
✅ Consistent security posture  
✅ Audit trails (Git history)  

### Disaster Recovery

✅ Rapid infrastructure recreation  
✅ Automated backups  
✅ Multi-region deployment  
✅ Reduced RTO/RPO  

---

## Declarative vs Imperative

### Imperative Approach

**How to do it**: Specify exact steps/commands to reach state

```bash
#!/bin/bash
# Create directory if not exists
if [ ! -d "/opt/myapp" ]; then
  mkdir /opt/myapp
fi

# Download application
curl -O https://example.com/myapp.tar.gz

# Extract
tar -xzf myapp.tar.gz -C /opt/myapp

# Start service
systemctl start myapp
```

**Pros**:
- Fine-grained control
- Useful for simple, linear tasks

**Cons**:
- Complex for large infrastructure
- Error-prone (order matters)
- Hard to determine current state
- Difficult to manage at scale

### Declarative Approach

**What you want**: Describe desired end state → Tool figures out how

```hcl
resource "aws_s3_bucket" "my_bucket" {
  bucket = "my-bucket-name"
  tags = {
    Name = "MyBucket"
  }
}
```

**Pros**:
- Simpler for complex systems
- Idempotent
- Easy to understand intent
- Tool optimizes execution

**Cons**:
- Less control over details
- Debugging can be complex
- Less flexible for edge cases

### Comparison

| Aspect | Imperative | Declarative |
|--------|-----------|-------------|
| **Approach** | How | What |
| **Complexity** | Lower for simple, higher for complex | Lower overall |
| **Idempotency** | Manual enforcement | Automatic |
| **State tracking** | Manual | Automatic |
| **Scalability** | Poor | Excellent |
| **Examples** | Bash scripts, Ansible playbooks | Terraform, CloudFormation |

---

## Popular IaC Tools

### Terraform (HashiCorp)

**Strengths**:
✅ Cloud-agnostic (AWS, Azure, GCP, Kubernetes, etc.)  
✅ Declarative HCL language  
✅ Robust state management  
✅ Extensible provider ecosystem  
✅ Large, active community  
✅ Excellent documentation  

**Best for**: Multi-cloud or hybrid environments

### AWS CloudFormation

**Strengths**:
✅ AWS-native integration  
✅ Change sets preview changes  
✅ Automatic rollback on failure  
✅ Managed service  

**Limitations**: AWS-only

### Azure Resource Manager (ARM)

**Strengths**:
✅ Azure-native integration  
✅ Multiple deployment scopes  
✅ Built-in functions  

**Limitations**: Azure-only, JSON syntax

### Google Cloud Deployment Manager

**Strengths**:
✅ GCP-native integration  
✅ Python/Jinja2 templating  

**Limitations**: GCP-only

### Configuration Management (Ansible, Chef, Puppet)

**Role**: Configure software on provisioned infrastructure

| Tool | Style | Agent | Use Case |
|------|-------|-------|----------|
| **Ansible** | Imperative/Procedural | No | Configuration management, orchestration |
| **Chef** | Imperative | Yes | Complex configurations |
| **Puppet** | Declarative | Yes | Large-scale automation |

**Common pattern**: Terraform (provision) + Ansible (configure)

### Why Terraform Dominates

| Reason | Value |
|--------|-------|
| **Multi-cloud** | Single tool for AWS, Azure, GCP, K8s |
| **HCL language** | Human-readable, expressive |
| **State management** | Intelligent diff and tracking |
| **Large ecosystem** | Thousands of modules available |
| **Community** | Massive ecosystem and support |

---

## Terraform Architecture

### 1. Providers: Cloud Platform Connectors

**Definition**: Plugins enabling Terraform to interact with specific APIs

**Purpose**: Translate Terraform code → cloud provider API calls

**Examples**:
- `aws`: AWS resources (EC2, S3, RDS, etc.)
- `azurerm`: Azure resources
- `google`: GCP resources
- `kubernetes`: K8s resources
- `github`: GitHub resources

**Configuration**:
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
```

### 2. Resources: Infrastructure Building Blocks

**Definition**: Infrastructure objects that Terraform creates, updates, deletes

**Syntax**: `resource "provider_type" "local_name" { configuration }`

**Components**:
- `provider_type`: Type of resource (aws_instance, azurerm_vm, etc.)
- `local_name`: Unique name in your configuration
- `configuration`: Desired attributes (key-value pairs)

**Example**:
```hcl
resource "aws_instance" "web_server" {
  ami           = "ami-0abcdef1234567890"
  instance_type = "t2.micro"
  
  tags = {
    Name        = "MyWebServer"
    Environment = "Production"
  }
}
```

**Referencing**: Use `resource_type.local_name` or `resource_type.local_name.attribute`

### 3. Data Sources: Query Existing Infrastructure

**Definition**: Read information about existing resources without managing them

**Purpose**: Reference external resources, query current infrastructure

**Syntax**: `data "provider_type" "local_name" { filters }`

**Example**:
```hcl
# Find latest Ubuntu AMI
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]  # Canonical
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-*"]
  }
}

# Use in resource
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
}
```

**Benefits**:
- Dynamic configurations
- Avoid hardcoding IDs
- Reference existing resources

---

## Terraform Workflow

### Standard Workflow

```
terraform init → terraform plan → terraform apply → (iterate) → terraform destroy
```

### 1. terraform init

**Purpose**: Initialize Terraform working directory

**Actions**:
- Download provider plugins
- Initialize backend configuration
- Download modules
- Create .terraform directory

**When to run**:
- First clone of project
- When adding/changing providers
- When switching backends

```bash
terraform init
```

### 2. terraform plan

**Purpose**: Preview changes (execution plan)

**Actions**:
- Read configuration
- Read current state
- Calculate differences
- Generate execution plan

**Output**: Clearly shows what will be created, updated, destroyed

**Critical for**: Safety and understanding impact

```bash
terraform plan
terraform plan -out=plan.tfplan  # Save plan for later use
```

### 3. terraform apply

**Purpose**: Execute plan and provision/update infrastructure

**Actions**:
- Generate plan if none provided
- Prompt for confirmation
- Execute API calls to providers
- Update state file

**Confirmation**: Required by default (use -auto-approve with caution)

```bash
terraform apply
terraform apply plan.tfplan  # Apply saved plan
terraform apply -auto-approve  # ⚠️ Skip confirmation
```

### 4. terraform destroy

**Purpose**: Remove all managed infrastructure

**Actions**:
- Generate destroy plan
- Prompt for confirmation
- Delete resources
- Remove from state

**Caution**: Destructive command - use carefully!

```bash
terraform destroy
terraform destroy -auto-approve  # ⚠️ No confirmation
```

### Workflow Example

**Step 1: Configuration**
```hcl
resource "aws_s3_bucket" "my_bucket" {
  bucket = "my-unique-bucket-12345"
  tags = {
    ManagedBy = "Terraform"
  }
}
```

**Step 2: Init**
```bash
$ terraform init
Terraform initialized in current directory
```

**Step 3: Plan**
```bash
$ terraform plan

Terraform will perform the following actions:
  # aws_s3_bucket.my_bucket will be created
  + resource "aws_s3_bucket" "my_bucket" {
      + bucket = "my-unique-bucket-12345"
      + tags   = { "ManagedBy" = "Terraform" }
    }

Plan: 1 to add, 0 to change, 0 to destroy.
```

**Step 4: Apply**
```bash
$ terraform apply
# ... plan output ...
Do you want to perform these actions? yes
aws_s3_bucket.my_bucket: Creating...
aws_s3_bucket.my_bucket: Creation complete
```

**Step 5: Subsequent Plan**
```bash
$ terraform plan

No changes. Your infrastructure matches the configuration.
```

**Step 6: Destroy**
```bash
$ terraform destroy
# ... destruction plan ...
Do you want to perform these actions? yes
aws_s3_bucket.my_bucket: Destroying...
aws_s3_bucket.my_bucket: Destruction complete
```

---

## State Management

### What is Terraform State?

**Definition**: Record of infrastructure that Terraform manages

**Storage**: JSON file (terraform.tfstate) - local by default, remote in production

**Contains**:
- Resource types and names
- Resource IDs (provider-specific)
- All resource attributes
- Resource dependencies
- Outputs and metadata

### Why State Matters

| Reason | Impact |
|--------|--------|
| **Mapping** | Links config to real infrastructure |
| **Tracking** | Knows what's been created |
| **Drift detection** | Finds manual infrastructure changes |
| **Planning** | Essential for plan command |
| **Collaboration** | Prevents conflicts in teams |

### Local vs Remote State

### Local State (Default)

**Location**: terraform.tfstate in working directory

**Suitable for**: Individual learning, tiny projects

**Drawbacks**:
❌ No team collaboration  
❌ Easy to lose/corrupt  
❌ No locking (concurrent access risk)  
❌ Manual backup responsibility  

### Remote State (Production)

**Backends**:
- AWS S3 (+ DynamoDB for locking)
- Azure Blob Storage
- Google Cloud Storage
- Terraform Cloud
- Consul

**Benefits**:
✅ Team collaboration  
✅ Centralized management  
✅ State locking (prevent conflicts)  
✅ Encryption at rest  
✅ Automatic backups  

### Configuring Remote Backend (S3)

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "global/s3/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

**Then**:
```bash
terraform init  # Migrate to remote backend
```

### State Locking

**Purpose**: Prevent concurrent modifications

**How it works**: 
- Lock acquired when running apply/destroy
- Prevents others from running until released
- Automatically released after operation completes

**Backends with locking**:
- S3 + DynamoDB
- Azure Blob Storage
- Google Cloud Storage
- Terraform Cloud

### State Commands

```bash
terraform state list          # List all resources
terraform state show aws_s3_bucket.my_bucket  # Show resource details
terraform state mv old new    # Rename resource
terraform state rm aws_s3_bucket.my_bucket    # Remove from state
```

### Important Notes

✅ **Never edit state file manually** - Use terraform state commands  
✅ **Always backup state** - It's your source of truth  
✅ **Encrypt sensitive data** - Use backend encryption  
✅ **Use state locking** - Prevents corruption  
✅ **Don't commit terraform.tfstate** - Add to .gitignore  

---

## Best Practices

### Configuration Management

✅ **Use version control** - Store all IaC in Git  
✅ **Pin provider versions** - Use `version = "~> 5.0"` constraints  
✅ **Document code** - Comments explaining complex logic  
✅ **Use remote state** - S3, Azure, or Terraform Cloud  
✅ **Enable state locking** - Prevent concurrent modifications  

### Planning & Deployment

✅ **Always run terraform plan first** - Preview changes before apply  
✅ **Review plan output carefully** - Understand what will happen  
✅ **Use CI/CD pipelines** - Automate apply process  
✅ **Implement approval gates** - Manual review before production  
✅ **Keep terraform apply logs** - For audit trails  

### Code Quality

✅ **Break into modules** - Reusable, maintainable code  
✅ **Use variables** - Parameterize configurations  
✅ **Validate syntax** - Run `terraform validate`  
✅ **Format code** - Use `terraform fmt`  
✅ **Follow naming conventions** - Clear, consistent names  

### Security

✅ **Never hardcode secrets** - Use environment variables, vaults  
✅ **Use IAM roles** - For provider authentication  
✅ **Encrypt state** - Enable encryption at rest  
✅ **Limit state access** - Restrict who can read/write  
✅ **Rotate credentials** - Regular credential updates  

### Team Collaboration

✅ **Use Terraform Cloud** - For teams and enterprises  
✅ **Enable state locking** - Prevent conflicts  
✅ **Code review process** - PR approval before merge  
✅ **Shared state backend** - Central source of truth  
✅ **Document architecture** - Keep README updated  

### Debugging & Maintenance

✅ **Enable debug logging** - `TF_LOG=DEBUG`  
✅ **Use terraform console** - Test expressions  
✅ **Run regular plans** - Detect drift  
✅ **Monitor state size** - Optimize large states  
✅ **Keep state clean** - Remove unused resources  

---

## Quick Reference

### Essential Commands

```bash
terraform init              # Initialize directory
terraform plan              # Generate execution plan
terraform apply             # Apply changes
terraform destroy           # Destroy infrastructure
terraform validate          # Check syntax
terraform fmt               # Format code
terraform console           # Interactive console
terraform state list        # List resources
terraform output            # Show outputs
```

### Common Workflow

```bash
# Initial setup
terraform init

# Before changes
terraform plan -out=plan.tfplan

# Apply changes
terraform apply plan.tfplan

# Check state
terraform state list

# Cleanup
terraform destroy
```

### Provider Configuration Patterns

```hcl
# AWS with region
provider "aws" {
  region = "us-east-1"
}

# Azure with subscription
provider "azurerm" {
  features {}
  subscription_id = var.subscription_id
}

# Kubernetes
provider "kubernetes" {
  host  = aws_eks_cluster.main.endpoint
  token = aws_eks_cluster_auth.main.token
}
```

---

## Common Mistakes to Avoid

❌ **Commit terraform.tfstate to Git**  
✅ Add `terraform.tfstate*` to `.gitignore`

❌ **Hardcode sensitive data**  
✅ Use environment variables or secrets manager

❌ **Skip terraform plan**  
✅ Always preview changes first

❌ **Use latest tags for providers**  
✅ Pin specific versions: `version = "~> 5.0"`

❌ **Store state locally for teams**  
✅ Use remote backend with locking

❌ **Apply without code review**  
✅ Implement approval process

---
