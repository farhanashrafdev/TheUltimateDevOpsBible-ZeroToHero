# Terraform Cheat Sheet - Complete Reference

## 🔧 Core Commands

### Initialization

```bash
# Initialize Terraform
terraform init
terraform init -upgrade            # Upgrade providers
terraform init -backend-config="key=value"  # Backend config
terraform init -migrate-state      # Migrate state
terraform init -reconfigure        # Reconfigure backend
```

### Planning

```bash
# Create execution plan
terraform plan
terraform plan -out=tfplan         # Save plan
terraform plan -var="key=value"    # With variable
terraform plan -var-file="prod.tfvars"  # Variable file
terraform plan -target=resource   # Specific resource
terraform plan -destroy            # Destroy plan
terraform plan -refresh=false      # Skip refresh
terraform plan -detailed-exitcode  # Detailed exit code
```

### Applying

```bash
# Apply changes
terraform apply
terraform apply -auto-approve      # Auto approve
terraform apply tfplan             # Use saved plan
terraform apply -target=resource   # Specific resource
terraform apply -parallelism=10    # Parallel operations
terraform apply -refresh-only      # Refresh only
```

### Destroying

```bash
# Destroy infrastructure
terraform destroy
terraform destroy -auto-approve    # Auto approve
terraform destroy -target=resource  # Specific resource
terraform destroy -var="key=value"  # With variable
```

### Validation

```bash
# Validate configuration
terraform validate
terraform fmt                      # Format code
terraform fmt -check               # Check formatting
terraform fmt -recursive           # Recursive format
terraform fmt -diff                # Show diffs
```

## 📦 State Management

### State Operations

```bash
# List resources
terraform state list
terraform state list -id=resource-id

# Show resource
terraform state show resource.address
terraform state show aws_instance.web

# Move resource
terraform state mv old.address new.address
terraform state mv aws_instance.old aws_instance.new

# Remove from state
terraform state rm resource.address
terraform state rm aws_instance.web

# Pull state
terraform state pull > state.json

# Push state
terraform state push state.json

# List state
terraform state list | grep pattern
```

### State Backends

```bash
# S3 backend
terraform {
  backend "s3" {
    bucket         = "terraform-state"
    key            = "path/to/state"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-lock"
  }
}

# Remote backend
terraform {
  backend "remote" {
    organization = "myorg"
    workspaces {
      name = "production"
    }
  }
}
```

## 🏷️ Workspaces

```bash
# List workspaces
terraform workspace list

# Create workspace
terraform workspace new production

# Select workspace
terraform workspace select production

# Show current workspace
terraform workspace show

# Delete workspace
terraform workspace delete staging
```

## 📝 Variables

### Variable Files

```bash
# Variable files
terraform.tfvars                   # Auto-loaded
*.auto.tfvars                     # Auto-loaded
terraform apply -var-file="prod.tfvars"

# Environment variables
export TF_VAR_instance_type=t3.medium
terraform apply
```

### Variable Definition

```hcl
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
  
  validation {
    condition     = can(regex("^t[23]\\..*", var.instance_type))
    error_message = "Must be t2 or t3 family."
  }
}

variable "tags" {
  description = "Resource tags"
  type        = map(string)
  default     = {}
}

variable "enabled" {
  description = "Enable resource"
  type        = bool
  default     = true
}
```

## 📤 Outputs

```bash
# Show outputs
terraform output
terraform output instance_id
terraform output -json             # JSON format
terraform output -raw instance_id  # Raw value
```

## 🔍 Inspection

### Resource Inspection

```bash
# Show resource
terraform show
terraform show resource.address
terraform show -json               # JSON output

# Graph
terraform graph | dot -Tsvg > graph.svg
terraform graph -type=plan
```

### Import

```bash
# Import resource
terraform import aws_instance.web i-1234567890abcdef0
terraform import 'aws_instance.web[0]' i-1234567890abcdef0

# Generate import block
terraform plan -generate-config-out=imported.tf
```

## 🏗️ Modules

### Using Modules

```hcl
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"
  
  name = "my-vpc"
  cidr = "10.0.0.0/16"
}
```

### Module Commands

```bash
# Get module
terraform get
terraform get -update              # Update modules

# Validate modules
terraform validate
```

## 🔐 Providers

### Provider Configuration

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
  
  default_tags {
    tags = {
      ManagedBy = "Terraform"
    }
  }
}
```

### Provider Commands

```bash
# Upgrade providers
terraform init -upgrade

# Lock providers
terraform providers lock
terraform providers lock -platform=linux_amd64
```

## 📊 Functions

### Common Functions

```hcl
# String functions
upper("hello")                     # "HELLO"
lower("HELLO")                     # "hello"
substr("hello", 0, 4)             # "hell"
split(",", "a,b,c")               # ["a", "b", "c"]
join(",", ["a", "b", "c"])        # "a,b,c"

# Numeric functions
max(1, 2, 3)                      # 3
min(1, 2, 3)                      # 1
abs(-5)                           # 5

# Collection functions
length(["a", "b"])                # 2
contains(["a", "b"], "a")         # true
keys({a=1, b=2})                  # ["a", "b"]
values({a=1, b=2})                # [1, 2]

# Type conversion
tostring(123)                     # "123"
tonumber("123")                   # 123
tobool("true")                    # true

# Date/time
timestamp()                       # Current timestamp
formatdate("YYYY-MM-DD", timestamp())

# File functions
file("path/to/file")              # Read file
fileexists("path/to/file")        # Check existence
templatefile("template.tpl", vars) # Template file

# Network functions
cidrhost("10.0.0.0/24", 5)        # "10.0.0.5"
cidrnetmask("10.0.0.0/24")        # "255.255.255.0"
```

## 🎯 Best Practices

### Code Organization

```bash
# Directory structure
.
├── main.tf                       # Main configuration
├── variables.tf                   # Variable definitions
├── outputs.tf                    # Output definitions
├── terraform.tfvars              # Variable values
├── backend.tf                     # Backend configuration
└── modules/                       # Reusable modules
    └── vpc/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

### Common Patterns

```hcl
# Locals
locals {
  common_tags = {
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
}

# Data sources
data "aws_ami" "latest" {
  most_recent = true
  owners      = ["amazon"]
}

# Conditional resources
resource "aws_instance" "web" {
  count = var.enabled ? 1 : 0
  # ...
}

# For loops
resource "aws_instance" "web" {
  for_each = var.instances
  # ...
}
```

## 🚨 Troubleshooting

### Common Issues

```bash
# State lock
terraform force-unlock LOCK_ID

# Refresh state
terraform refresh
terraform apply -refresh-only

# Validate syntax
terraform validate

# Check formatting
terraform fmt -check -recursive

# Debug
export TF_LOG=DEBUG
terraform apply
```

### State Issues

```bash
# List state
terraform state list

# Show state
terraform state show resource

# Move resource
terraform state mv old new

# Remove from state
terraform state rm resource
```

## 📚 Quick Reference

### File Extensions

```
.tf        # Terraform configuration
.tfvars    # Variable values
.tfstate   # State file
.tfstate.backup  # State backup
.terraform/      # Provider cache
```

### Common Resources

```hcl
# AWS
resource "aws_instance" "web" { }
resource "aws_s3_bucket" "data" { }
resource "aws_vpc" "main" { }

# Azure
resource "azurerm_resource_group" "main" { }
resource "azurerm_virtual_network" "main" { }

# GCP
resource "google_compute_instance" "web" { }
resource "google_storage_bucket" "data" { }
```

---

**Remember**: Always run `terraform plan` before `terraform apply`. Use version control for all Terraform files. Keep state files secure and use remote backends.
