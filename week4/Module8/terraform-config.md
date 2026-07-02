# Terraform HCL & Providers - Module 8 Summary

## Overview
This module covers the foundational concepts of Infrastructure as Code (IaC) using Terraform, focusing on HashiCorp Configuration Language (HCL) syntax, cloud provider configuration, and resource management.


## Part 1: HashiCorp Configuration Language (HCL) Fundamentals

### 1.1 HCL Blocks - Structural Foundation

Blocks are the core building blocks of HCL configurations. Structure:

```hcl
block_type "label1" "label2" {
  # configuration arguments
}
```

**Common Block Types:**
- `provider` - Configure cloud provider access
- `resource` - Define infrastructure components
- `variable` - Declare input parameters
- `output` - Export configuration values
- `data` - Query existing infrastructure
- `locals` - Define reusable local values
- `module` - Reference reusable configurations

### 1.2 HCL Arguments - Key-Value Pairs

Arguments define specific configuration values within blocks using `key = value` syntax.

**Supported Data Types:**
- **Strings**: `"example"` - Enclosed in quotes
- **Numbers**: `80`, `3.14` - Integers or floats
- **Booleans**: `true`, `false` - Boolean values
- **Lists**: `["item1", "item2"]` - Ordered collections
- **Maps**: `{ key1 = "value1", key2 = "value2" }` - Key-value pairs
- **Sets**: `{ "item1", "item2" }` - Unique unordered values
- **Tuples**: `["string", 123, true]` - Mixed type collections

**Example:**
```hcl
provider "aws" {
  region     = "us-east-1"      # String
  access_key = "YOUR_KEY"       # String
  tags       = { env = "prod" } # Map
}
```

### 1.3 HCL Expressions - Dynamic Values

Expressions enable dynamic value generation, references, and conditional logic.

#### Reference Values From:
```hcl
var.instance_count              # Variable reference
aws_instance.web_server.id      # Resource attribute
data.aws_ami.latest.id          # Data source attribute
local.common_tags               # Local value
output.web_server_ip            # Output reference
```

#### Built-in Functions:
```hcl
upper("hello")                  # Returns "HELLO"
length(["a", "b", "c"])         # Returns 3
merge({a=1}, {b=2})            # Returns {a=1, b=2}
```

#### Conditional Expressions:
```hcl
environment = var.is_production ? "prod" : "dev"
```

#### Splat Expressions:
```hcl
# Extract specific attribute from list of resources
aws_instance.web_servers[*].public_ip
```

#### String Interpolation:
```hcl
bucket = "my-bucket-${random_string.suffix.result}"
```

---

## Part 2: Terraform Providers

### 2.1 What Are Providers?

Providers are plugins that enable Terraform to interact with specific cloud platforms and services. They translate Terraform's generic resource definitions into API calls for the target platform.

### 2.2 Major Cloud Providers

| Provider | Use Case | Key Resources |
|----------|----------|---------------|
| **AWS** | Amazon Web Services | EC2, S3, VPC, RDS |
| **Azure** (azurerm) | Microsoft Azure | VMs, Storage Accounts, Virtual Networks |
| **GCP** (google) | Google Cloud Platform | Compute Engine, Cloud Storage, GKE |

### 2.3 Provider Configuration

#### Basic Syntax:
```hcl
provider "aws" {
  region = "us-east-1"
}
```

#### Pin Provider Versions:
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"        # ~> means compatible versions
    }
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "= 3.0.0"       # = means exact version
    }
    google = {
      source  = "hashicorp/google"
      version = ">= 4.0, < 5.0" # Range constraint
    }
  }
}
```

#### Multiple Provider Instances (Aliases):
```hcl
provider "aws" {
  alias  = "us-east-1"
  region = "us-east-1"
}

provider "aws" {
  alias  = "eu-west-2"
  region = "eu-west-2"
}

# Use aliased providers in resources:
resource "aws_instance" "web_us" {
  provider = aws.us-east-1
  # ... configuration ...
}
```

### 2.4 Credential Configuration

#### AWS Credentials (Priority Order):
1. Environment variables: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`
2. Shared credentials file: `~/.aws/credentials`
3. Shared config file: `~/.aws/config`
4. EC2/ECS/EKS IAM role (when running on AWS resources)

**Best Practice**: Use environment variables or IAM roles, not hardcoded credentials.

#### Azure Credentials:
- Environment variables: `ARM_CLIENT_ID`, `ARM_CLIENT_SECRET`, `ARM_TENANT_ID`, `ARM_SUBSCRIPTION_ID`
- Azure CLI authentication: `az login`

#### GCP Credentials:
- Environment variable: `GOOGLE_APPLICATION_CREDENTIALS` (path to service account JSON key)
- Application Default Credentials (ADC) for GCP resources
- Service account credentials

---

## Part 3: Terraform Resources

### 3.1 What Are Resources?

Resources represent managed infrastructure objects. When defined in configuration, Terraform ensures the actual cloud resource matches the declared state.

### 3.2 Resource Block Syntax

```hcl
resource "resource_type" "local_name" {
  argument1 = "value1"
  argument2 = "value2"
}
```

**Components:**
- `resource_type` - Provider-specific identifier (e.g., `aws_instance`, `aws_s3_bucket`)
- `local_name` - Unique name within your configuration for referencing
- Arguments - Configuration settings for the resource

### 3.3 Common Resource Types

**AWS Examples:**
- `aws_instance` - EC2 virtual machine
- `aws_s3_bucket` - S3 storage bucket
- `aws_vpc` - Virtual Private Cloud
- `aws_subnet` - VPC subnet
- `aws_security_group` - Firewall rules
- `aws_rds_instance` - Managed database

### 3.4 Referencing Resource Attributes

Access resource attributes using: `resource_type.local_name.attribute`

```hcl
resource "aws_instance" "web_server" {
  ami           = "ami-0abcdef1234567890"
  instance_type = "t2.micro"
}

output "server_ip" {
  value = aws_instance.web_server.public_ip  # Reference attribute
}
```

### 3.5 Example: S3 Bucket Resource

```hcl
resource "aws_s3_bucket" "my_app_data" {
  bucket = "my-unique-bucket-name"
  acl    = "private"

  tags = {
    Environment = "Development"
    Project     = "MyApp"
  }
}

output "bucket_name" {
  value = aws_s3_bucket.my_app_data.bucket
}

output "bucket_arn" {
  value = aws_s3_bucket.my_app_data.arn
}
```

---

## Part 4: Data Sources

### 4.1 What Are Data Sources?

Data sources fetch information about existing infrastructure without managing it. Useful for querying resources created outside Terraform or dynamically selecting values.

### 4.2 Data Source Block Syntax

```hcl
data "data_source_type" "local_name" {
  filter_argument = "value"
}
```

### 4.3 Common Use Cases

- **Reference existing resources**: Find VPC, subnet, or security group IDs
- **Dynamic configuration**: Look up latest AMI ID or DNS zone
- **Information gathering**: Audit and reporting
- **Cross-configuration dependencies**: Reference resources from different Terraform configs

### 4.4 AWS Data Source Examples

```hcl
data "aws_ami" "ubuntu" {          # Find latest Ubuntu AMI
data "aws_vpcs" "available"        # List existing VPCs
data "aws_subnet" "selected"       # Find specific subnet
data "aws_caller_identity" "self"  # Get current AWS account info
```

### 4.5 Example: Fetch Latest Ubuntu AMI

```hcl
data "aws_ami" "ubuntu" {
  most_recent = true

  filter {
    name   = "owner-alias"
    values = ["amazon"]
  }

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
}

# Use the AMI ID in an instance:
resource "aws_instance" "web_server" {
  ami           = data.aws_ami.ubuntu.id    # Reference data source attribute
  instance_type = "t2.micro"

  tags = {
    Name = "MyUbuntuWebServer"
  }
}
```

---

## Part 5: Variables & Outputs

### 5.1 Terraform Variables (Input Parameters)

Variables allow external values to be passed into configurations, enabling reusability across environments.

#### Declare Variables:
```hcl
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
  # sensitive  = true           # Hide in output
}

variable "instance_count" {
  description = "Number of instances"
  type        = number
  default     = 1
}

variable "tags" {
  description = "Resource tags"
  type        = map(string)
  default = {
    Environment = "Dev"
  }
}
```

**Variable Attributes:**
- `description` - Documentation of the variable
- `type` - Data type (string, number, bool, list, map, etc.)
- `default` - Optional default value
- `sensitive` - Mark as sensitive to hide in logs
- `validation` - Define validation rules

#### Reference Variables:
```hcl
resource "aws_instance" "web" {
  instance_type = var.instance_type
  tags          = var.tags
}
```

### 5.2 Ways to Provide Variable Values

#### 1. Command-Line Flags:
```bash
terraform apply -var="instance_type=t3.medium" -var="region=eu-west-1"
```

#### 2. Variable Definition Files (.tfvars):
```hcl
# dev.tfvars
instance_type  = "t2.micro"
region         = "us-east-1"
instance_count = 2
tags = {
  Environment = "Development"
  Project     = "MyApp"
}
```

```bash
terraform apply -var-file="dev.tfvars"
```

#### 3. Environment Variables:
```bash
export TF_VAR_instance_type="t3.small"
export TF_VAR_region="ap-southeast-2"
terraform apply
```

#### 4. Default Values:
Automatically used if no value is provided.

#### 5. Interactive Prompts:
Terraform prompts for input if variable has no default.

### 5.3 Terraform Outputs (Export Values)

Outputs expose values after infrastructure is created, useful for sharing information with other tools or configurations.

#### Declare Outputs:
```hcl
output "instance_public_ip" {
  description = "Public IP of web server"
  value       = aws_instance.web_server.public_ip
  # sensitive  = true  # Hide sensitive outputs
}

output "bucket_name" {
  description = "Name of created S3 bucket"
  value       = aws_s3_bucket.my_app_data.bucket
}
```

**Output Attributes:**
- `description` - Explanation of the output
- `value` - Expression generating the output value
- `sensitive` - Mask sensitive output values
- `depends_on` - Explicit dependencies

#### View Outputs:
```bash
terraform output instance_public_ip    # Get specific output
terraform output                       # Show all outputs
```

---

## Part 6: Complete Example Configuration

### Project Structure:
```
terraform-aws-provider-lab/
├── versions.tf
├── main.tf
├── variables.tf
├── outputs.tf
├── dev.tfvars
└── prod.tfvars
```

### versions.tf:
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

### main.tf:
```hcl
provider "aws" {
  region = "us-east-1"
}

resource "random_string" "bucket_suffix" {
  length  = 16
  special = false
  upper   = false
}

resource "aws_s3_bucket" "example_bucket" {
  bucket = "${var.bucket_name_prefix}-${random_string.bucket_suffix.result}"
  acl    = var.bucket_acl

  tags = merge(var.bucket_tags, {
    Name = "Terraform Example Bucket"
  })
}
```

### variables.tf:
```hcl
variable "bucket_name_prefix" {
  description = "Prefix for S3 bucket name"
  type        = string
  default     = "my-app-bucket"
}

variable "bucket_acl" {
  description = "S3 bucket ACL"
  type        = string
  default     = "private"
  
  validation {
    condition     = contains(["private", "public-read", "public-read-write"], var.bucket_acl)
    error_message = "Must be a valid S3 ACL."
  }
}

variable "bucket_tags" {
  description = "Tags for the bucket"
  type        = map(string)
  default = {
    Environment = "Dev"
    ManagedBy   = "Terraform"
  }
}
```

### outputs.tf:
```hcl
output "bucket_name" {
  description = "Name of the S3 bucket"
  value       = aws_s3_bucket.example_bucket.bucket
}

output "bucket_arn" {
  description = "ARN of the S3 bucket"
  value       = aws_s3_bucket.example_bucket.arn
}
```

### dev.tfvars:
```hcl
bucket_name_prefix = "my-dev-app-bucket"
bucket_acl         = "private"
bucket_tags = {
  Environment = "Development"
  Project     = "MyApp"
  Owner       = "DevTeam"
}
```

---

## Part 7: Practical Workflow

### Initialize Project:
```bash
terraform init
```
Downloads provider plugins and initializes the working directory.

### Review Changes (Plan):
```bash
terraform plan -var-file="dev.tfvars"
```
Shows what Terraform intends to create, update, or destroy.

### Apply Configuration:
```bash
terraform apply -var-file="dev.tfvars"
```
Provisions the infrastructure. Requires confirmation (type `yes`).

### View Outputs:
```bash
terraform output
```
Displays all declared output values.

### Destroy Infrastructure (when done):
```bash
terraform destroy -var-file="dev.tfvars"
```
Removes all managed resources (use carefully in production).

---

## Quick Reference

| Task | Command |
|------|---------|
| Initialize | `terraform init` |
| Validate | `terraform validate` |
| Plan | `terraform plan -var-file="dev.tfvars"` |
| Apply | `terraform apply -var-file="dev.tfvars"` |
| View Output | `terraform output` |
| Destroy | `terraform destroy` |
| Format | `terraform fmt` |

---
