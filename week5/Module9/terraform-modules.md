# Terraform Modules

## Overview

Terraform modules are reusable, self-contained packages of Terraform configurations that encapsulate related infrastructure resources. They promote code reusability, organization, and maintainability through abstraction and composition.

---

## Table of Contents

1. [What are Modules](#what-are-modules)
2. [Benefits](#benefits)
3. [Module Structure](#module-structure)
4. [Creating Modules](#creating-modules)
5. [Module Block & Sources](#module-block--sources)
6. [Passing Variables](#passing-variables)
7. [Module Outputs](#module-outputs)
8. [Best Practices](#best-practices)
9. [Common Patterns](#common-patterns)

---

## What are Modules

### Definition

**Terraform modules** are directories containing `.tf` files that define a set of related infrastructure resources, functioning like functions or packages in programming languages.

**Core Concept**: "Write once, use many times"

### Key Characteristics

✅ Self-contained infrastructure packages  
✅ Reusable across projects and environments  
✅ Encapsulate complexity  
✅ Promote consistency  
✅ Enable team collaboration  
✅ Simplify large deployments  

### Module vs Non-Module

| Aspect | Without Modules | With Modules |
|--------|-----------------|--------------|
| **Code reuse** | Copy-paste | Single reference |
| **Maintenance** | Update in multiple places | Update once |
| **Complexity** | Monolithic files | Organized units |
| **Scalability** | Difficult for large projects | Excellent |
| **Consistency** | Manual enforcement | Built-in |

---

## Benefits

### 1. Reusability

Deploy same infrastructure pattern across:
- Multiple environments (dev, staging, prod)
- Different projects
- Various regions

**Example**: Deploy same VPC module in 5 regions with single definition

### 2. Organization

Break down complex infrastructure:
- **Without modules**: 1000+ lines in main.tf
- **With modules**: Organized vpc/, security_groups/, database/ modules

### 3. Maintainability

Update infrastructure pattern once:
- Changes propagate automatically
- Centralized source of truth
- Reduces errors and inconsistencies

### 4. Abstraction

Module users don't need to know:
- Implementation details
- Resource interdependencies
- Complex configuration logic

Only need to understand inputs/outputs

### 5. Consistency

Standardized infrastructure:
- Same patterns across org
- Enforced security/compliance
- Predictable resource configuration

### 6. Collaboration

Enable team workflows:
- Platform teams build modules
- Application teams consume modules
- Clear interfaces and contracts

### 7. Testability

Smaller, focused modules are:
- Easier to test
- Faster to validate
- More reliable

---

## Module Structure

### Directory Layout

```
my-terraform-project/
├── main.tf
├── variables.tf
├── outputs.tf
├── terraform.tfvars
├── .gitignore
└── modules/
    ├── vpc/
    │   ├── main.tf
    │   ├── variables.tf
    │   ├── outputs.tf
    │   └── README.md
    ├── security_groups/
    │   ├── main.tf
    │   ├── variables.tf
    │   ├── outputs.tf
    │   └── README.md
    └── database/
        ├── main.tf
        ├── variables.tf
        ├── outputs.tf
        └── README.md
```

### Essential Module Files

| File | Purpose |
|------|---------|
| **main.tf** | Resource definitions |
| **variables.tf** | Input variables |
| **outputs.tf** | Exported values |
| **README.md** | Documentation (critical) |
| **.gitignore** | Version control excludes |

### Minimal Module

```hcl
# modules/example/main.tf
resource "aws_s3_bucket" "example" {
  bucket = var.bucket_name
}

# modules/example/variables.tf
variable "bucket_name" {
  type = string
}

# modules/example/outputs.tf
output "bucket_id" {
  value = aws_s3_bucket.example.id
}
```

---

## Creating Modules

### Step 1: Create Directory Structure

```bash
mkdir -p modules/vpc
cd modules/vpc
touch main.tf variables.tf outputs.tf README.md
```

### Step 2: Define Resources (main.tf)

```hcl
# modules/vpc/main.tf

resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr_block
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name        = "${var.environment}-vpc"
    Environment = var.environment
  }
}

resource "aws_internet_gateway" "gw" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "${var.environment}-igw"
  }
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.gw.id
  }

  tags = {
    Name = "${var.environment}-public-rt"
  }
}
```

### Step 3: Define Inputs (variables.tf)

```hcl
# modules/vpc/variables.tf

variable "vpc_cidr_block" {
  description = "The CIDR block for the VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "environment" {
  description = "Environment name (dev, staging, prod)"
  type        = string
  default     = "dev"

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}
```

### Step 4: Define Outputs (outputs.tf)

```hcl
# modules/vpc/outputs.tf

output "vpc_id" {
  description = "The ID of the created VPC"
  value       = aws_vpc.main.id
}

output "vpc_cidr_block" {
  description = "The CIDR block of the VPC"
  value       = aws_vpc.main.cidr_block
}

output "route_table_id" {
  description = "The ID of the public route table"
  value       = aws_route_table.public.id
}
```

### Step 5: Document (README.md)

```markdown
# VPC Module

Creates an AWS VPC with Internet Gateway and public route table.

## Usage

```hcl
module "my_vpc" {
  source = "./modules/vpc"

  vpc_cidr_block = "10.1.0.0/16"
  environment    = "production"
}
```

## Inputs

| Name | Type | Default | Description |
|------|------|---------|-------------|
| vpc_cidr_block | string | 10.0.0.0/16 | VPC CIDR block |
| environment | string | dev | Environment name |

## Outputs

| Name | Description |
|------|-------------|
| vpc_id | VPC ID |
| vpc_cidr_block | VPC CIDR block |
| route_table_id | Public route table ID |
```

---

## Module Block & Sources

### Module Block Syntax

```hcl
module "local_name" {
  source = "source_location"

  # Pass values to module input variables
  variable_name = value
  another_var   = var.root_variable
}

# Access module outputs
resource "aws_instance" "web" {
  subnet_id = module.local_name.subnet_id
}

# Export module output
output "vpc_id" {
  value = module.local_name.vpc_id
}
```

### Source Types

#### 1. Local Path (Same Project)

```hcl
module "network" {
  source = "./modules/vpc"

  vpc_cidr_block = "10.0.0.0/16"
  environment    = "dev"
}
```

**When to use**:
- Development and testing
- Tightly coupled components
- Single project ownership

**Pros**: Simple, immediate changes reflected
**Cons**: Not reusable across projects

#### 2. Terraform Registry

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"

  name = "prod-vpc"
  cidr = "10.0.0.0/16"

  azs            = ["us-east-1a", "us-east-1b"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24"]

  enable_nat_gateway = true
  single_nat_gateway = true

  tags = {
    Environment = "production"
  }
}
```

**Format**: `namespace/name/provider`

**Popular modules**:
- `terraform-aws-modules/vpc/aws`
- `terraform-aws-modules/eks/aws`
- `terraform-aws-modules/rds/aws`

**Pros**: 
- Battle-tested
- Well-documented
- Community-maintained
- Versioned

**Cons**: 
- External dependency
- Less customization
- Trust requirements

#### 3. Git Repository

```hcl
# Public repository
module "my_module" {
  source = "git::https://github.com/my-org/modules.git//vpc?ref=v1.0.0"
  
  # Or using SSH
  # source = "git::ssh://git@github.com/my-org/modules.git//vpc?ref=v1.0.0"
}
```

**URL Format**:
```
git::<protocol>://<host>/<path>//<module_path>?ref=<ref>
```

**Parameters**:
- `protocol`: https, ssh, git
- `host`: github.com, gitlab.com, etc.
- `path`: Repository path
- `module_path`: Module subdirectory (after //)
- `ref`: Branch, tag, or commit SHA

**Pros**:
- Organizational control
- Full versioning
- Git workflow
- Private repos supported

**Cons**:
- Requires Git setup
- Authentication needed
- More complex

### Accessing Registry Modules

```bash
# Visit registry
https://registry.terraform.io/

# Search for modules
# Example: "kubernetes", "database", "network"

# Find module page
# Review: inputs, outputs, examples, requirements
```

---

## Passing Variables

### Basic Variable Passing

```hcl
# Module definition
variable "instance_count" {
  type = number
}

# Root usage
module "app" {
  source = "./modules/app"
  
  instance_count = 3
}
```

### Using Root Variables

```hcl
# Root variables.tf
variable "environment" {
  type = string
}

# Root main.tf
module "network" {
  source = "./modules/vpc"
  
  environment = var.environment  # Pass root var to module
}
```

### Complex Types

```hcl
# List
module "servers" {
  source = "./modules/servers"
  
  instance_names = ["web-01", "web-02", "web-03"]
}

# Map
module "tags" {
  source = "./modules/resources"
  
  tags = {
    Environment = "prod"
    Owner       = "DevOps"
    CostCenter  = "12345"
  }
}

# Object
module "database" {
  source = "./modules/database"
  
  db_config = {
    engine               = "postgres"
    instance_class       = "db.t3.small"
    allocated_storage    = 100
  }
}
```

### Variable Precedence

1. **Explicit module argument** (highest priority)
2. **tfvars file or -var flag** (in root config)
3. **Module variable default** (lowest priority)

```hcl
# Example: Which value wins?

# Module default
variable "region" {
  default = "us-east-1"
}

# tfvars file
# region = "us-west-2"

# Module call
module "app" {
  source = "./modules/app"
  region = "eu-west-1"  # THIS WINS (explicit argument)
}
```

---

## Module Outputs

### Defining Outputs

```hcl
# modules/vpc/outputs.tf

output "vpc_id" {
  description = "The VPC ID"
  value       = aws_vpc.main.id
}

output "subnets" {
  description = "List of subnet IDs"
  value       = [for subnet in aws_subnet.public : subnet.id]
}

output "endpoints" {
  description = "Service endpoints"
  value = {
    nat_gateway_ip = aws_nat_gateway.main.public_ip
    igw_id         = aws_internet_gateway.gw.id
  }
  sensitive = true  # Mask in output
}
```

### Accessing Module Outputs

```hcl
# Root configuration

output "network_vpc_id" {
  value = module.network.vpc_id
}

# Using in resources
resource "aws_security_group" "app" {
  vpc_id = module.network.vpc_id
  # ...
}

# Chaining modules
module "database" {
  source = "./modules/database"
  
  vpc_id = module.network.vpc_id  # Output from network module
}
```

### Module Output Types

```hcl
# Simple value
output "instance_id" {
  value = aws_instance.web.id
}

# List of values
output "public_ips" {
  value = aws_instance.web[*].public_ip
}

# Map of values
output "resource_ids" {
  value = {
    vpc    = aws_vpc.main.id
    subnet = aws_subnet.public.id
  }
}

# Complex object
output "database_info" {
  value = {
    endpoint = aws_db_instance.main.endpoint
    port     = aws_db_instance.main.port
    name     = aws_db_instance.main.db_name
  }
  sensitive = true
}
```

---

## Best Practices

### 1. Single Responsibility

```hcl
# ❌ DON'T: Module does too much
module "everything" {
  # Creates VPC, databases, clusters, etc.
}

# ✅ DO: Focused modules
module "vpc" { }
module "database" { }
module "cluster" { }
```

### 2. Clear Naming

```hcl
# ❌ BAD: Unclear names
variable "val1" { }
variable "cfg" { }

# ✅ GOOD: Descriptive names
variable "instance_type" { }
variable "database_config" { }
variable "enable_high_availability" { }
```

### 3. Sensible Defaults

```hcl
# ✅ Provide defaults for common cases
variable "instance_type" {
  type    = string
  default = "t3.micro"
}

variable "environment" {
  type    = string
  default = "development"
}
```

### 4. Thorough Documentation

```hcl
# Module README.md must include:
# - Purpose and description
# - All inputs with descriptions
# - All outputs
# - Usage examples
# - Prerequisites
# - Troubleshooting
```

### 5. Version Modules

```hcl
# ✅ Always pin versions
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.1.0"
}

# Git with tags
module "network" {
  source = "git::https://github.com/org/modules.git//vpc?ref=v1.2.0"
}
```

### 6. Avoid Hardcoding

```hcl
# ❌ BAD: Hardcoded values
resource "aws_instance" "web" {
  ami           = "ami-0abcdef1234567890"
  instance_type = "t2.micro"
}

# ✅ GOOD: Use variables
variable "ami_id" { type = string }
variable "instance_type" { type = string }

resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = var.instance_type
}
```

### 7. Expose Necessary Outputs

```hcl
# ✅ Export important information
output "instance_id" {
  value = aws_instance.web.id
}

output "public_ip" {
  value = aws_instance.web.public_ip
}

output "security_group_id" {
  value = aws_security_group.web.id
}
```

### 8. Design for Composition

```hcl
# ✅ Build modules that work together
module "vpc" {
  source = "./modules/vpc"
}

module "security" {
  source = "./modules/security"
  vpc_id = module.vpc.vpc_id
}

module "compute" {
  source = "./modules/compute"
  vpc_id = module.vpc.vpc_id
  sg_id  = module.security.sg_id
}
```

---

## Common Patterns

### Multiple Environments

```hcl
# prod.tfvars
environment = "production"
instance_count = 3
instance_type = "t3.large"

# dev.tfvars
environment = "development"
instance_count = 1
instance_type = "t2.micro"

# Apply with different files
terraform apply -var-file="prod.tfvars"
terraform apply -var-file="dev.tfvars"
```

### Multi-Region Deployment

```hcl
module "vpc_us_east" {
  source = "./modules/vpc"
  
  region            = "us-east-1"
  vpc_cidr_block    = "10.0.0.0/16"
  environment       = "production"
}

module "vpc_eu_west" {
  source = "./modules/vpc"
  
  region            = "eu-west-1"
  vpc_cidr_block    = "10.1.0.0/16"
  environment       = "production"
}
```

### Module Composition

```hcl
# Root main.tf - orchestrates modules

module "network" {
  source = "./modules/vpc"
}

module "security" {
  source = "./modules/security"
  vpc_id = module.network.vpc_id
}

module "compute" {
  source = "./modules/compute"
  vpc_id = module.network.vpc_id
  sg_id  = module.security.sg_id
}

module "database" {
  source = "./modules/database"
  vpc_id = module.network.vpc_id
}

# Root outputs aggregate module outputs
output "infrastructure" {
  value = {
    vpc_id = module.network.vpc_id
    security_groups = module.security.sg_ids
    instance_ids = module.compute.instance_ids
    db_endpoint = module.database.endpoint
  }
}
```

### Conditional Module Usage

```hcl
variable "enable_database" {
  type    = bool
  default = true
}

# Conditionally create module
module "database" {
  count  = var.enable_database ? 1 : 0
  source = "./modules/database"
  
  vpc_id = module.network.vpc_id
}

# Reference conditional output
output "db_endpoint" {
  value = try(module.database[0].endpoint, null)
}
```

---

## Quick Reference

### Creating a Module

```bash
mkdir -p modules/mymodule
cd modules/mymodule
touch main.tf variables.tf outputs.tf README.md
```

### Module Skeleton

```hcl
# main.tf - Resources
resource "aws_resource" "example" {
  argument = var.variable_name
}

# variables.tf - Inputs
variable "variable_name" {
  type = string
  description = "Description"
}

# outputs.tf - Outputs
output "output_name" {
  value = aws_resource.example.id
}
```

### Using a Module

```hcl
# Local
module "local" {
  source = "./modules/mymodule"
  var = value
}

# Registry
module "registry" {
  source  = "namespace/name/provider"
  version = "1.0.0"
  var = value
}

# Git
module "git" {
  source = "git::https://github.com/org/repo.git//path?ref=v1.0.0"
  var = value
}
```

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Module not found | Verify source path, run `terraform init` |
| Variable not recognized | Check module variables.tf spelling |
| Output not available | Verify output is defined in outputs.tf |
| Git authentication fails | Configure SSH keys or Git credentials |
| Module version mismatch | Update version constraint or pin specific version |

---
