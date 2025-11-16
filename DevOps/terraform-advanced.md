# Terraform Advanced: Complete Advanced Patterns Guide

## 🎯 Introduction

This guide covers advanced Terraform patterns, techniques, and best practices for managing complex infrastructure at scale. Master these to become a Terraform expert.

### Advanced Topics Covered

- Modules and module composition
- Workspaces and environment management
- Dynamic blocks and for expressions
- State management and remote backends
- Terraform Cloud/Enterprise
- Advanced functions and expressions
- Testing and validation
- Performance optimization

## 🏗️ Advanced Modules

### Module Structure

```
modules/
  vpc/
    main.tf
    variables.tf
    outputs.tf
    versions.tf
    README.md
    examples/
      basic/
        main.tf
```

### Complex Module Example

**modules/vpc/main.tf**:
```hcl
terraform {
  required_version = ">= 1.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

resource "aws_vpc" "this" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = var.enable_dns_hostnames
  enable_dns_support   = var.enable_dns_support
  
  tags = merge(
    var.tags,
    {
      Name = var.name
    }
  )
}

resource "aws_internet_gateway" "this" {
  count  = var.create_igw ? 1 : 0
  vpc_id = aws_vpc.this.id
  
  tags = merge(
    var.tags,
    {
      Name = "${var.name}-igw"
    }
  )
}

resource "aws_subnet" "public" {
  count = length(var.public_subnets)
  
  vpc_id                  = aws_vpc.this.id
  cidr_block              = var.public_subnets[count.index]
  availability_zone       = var.availability_zones[count.index]
  map_public_ip_on_launch = var.map_public_ip_on_launch
  
  tags = merge(
    var.tags,
    {
      Name = "${var.name}-public-${count.index + 1}"
      Type = "Public"
    }
  )
}

resource "aws_subnet" "private" {
  count = length(var.private_subnets)
  
  vpc_id            = aws_vpc.this.id
  cidr_block        = var.private_subnets[count.index]
  availability_zone = var.availability_zones[count.index]
  
  tags = merge(
    var.tags,
    {
      Name = "${var.name}-private-${count.index + 1}"
      Type = "Private"
    }
  )
}

resource "aws_nat_gateway" "this" {
  count = var.create_nat_gateway ? length(var.public_subnets) : 0
  
  allocation_id = aws_eip.nat[count.index].id
  subnet_id     = aws_subnet.public[count.index].id
  
  tags = merge(
    var.tags,
    {
      Name = "${var.name}-nat-${count.index + 1}"
    }
  )
  
  depends_on = [aws_internet_gateway.this]
}

resource "aws_eip" "nat" {
  count = var.create_nat_gateway ? length(var.public_subnets) : 0
  
  domain = "vpc"
  
  tags = merge(
    var.tags,
    {
      Name = "${var.name}-nat-eip-${count.index + 1}"
    }
  )
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.this.id
  
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.this[0].id
  }
  
  tags = merge(
    var.tags,
    {
      Name = "${var.name}-public-rt"
    }
  )
}

resource "aws_route_table" "private" {
  count = length(var.private_subnets)
  
  vpc_id = aws_vpc.this.id
  
  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.this[count.index].id
  }
  
  tags = merge(
    var.tags,
    {
      Name = "${var.name}-private-rt-${count.index + 1}"
    }
  )
}

resource "aws_route_table_association" "public" {
  count = length(var.public_subnets)
  
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "private" {
  count = length(var.private_subnets)
  
  subnet_id      = aws_subnet.private[count.index].id
  route_table_id = aws_route_table.private[count.index].id
}
```

**modules/vpc/variables.tf**:
```hcl
variable "name" {
  description = "Name of the VPC"
  type        = string
}

variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string
  validation {
    condition     = can(cidrhost(var.vpc_cidr, 0))
    error_message = "Must be a valid CIDR block."
  }
}

variable "availability_zones" {
  description = "List of availability zones"
  type        = list(string)
}

variable "public_subnets" {
  description = "List of public subnet CIDR blocks"
  type        = list(string)
}

variable "private_subnets" {
  description = "List of private subnet CIDR blocks"
  type        = list(string)
}

variable "create_igw" {
  description = "Create internet gateway"
  type        = bool
  default     = true
}

variable "create_nat_gateway" {
  description = "Create NAT gateways"
  type        = bool
  default     = true
}

variable "map_public_ip_on_launch" {
  description = "Map public IP on launch for public subnets"
  type        = bool
  default     = true
}

variable "enable_dns_hostnames" {
  description = "Enable DNS hostnames"
  type        = bool
  default     = true
}

variable "enable_dns_support" {
  description = "Enable DNS support"
  type        = bool
  default     = true
}

variable "tags" {
  description = "Tags to apply to resources"
  type        = map(string)
  default     = {}
}
```

**modules/vpc/outputs.tf**:
```hcl
output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.this.id
}

output "vpc_cidr_block" {
  description = "CIDR block of the VPC"
  value       = aws_vpc.this.cidr_block
}

output "public_subnet_ids" {
  description = "List of public subnet IDs"
  value       = aws_subnet.public[*].id
}

output "private_subnet_ids" {
  description = "List of private subnet IDs"
  value       = aws_subnet.private[*].id
}

output "nat_gateway_ids" {
  description = "List of NAT gateway IDs"
  value       = aws_nat_gateway.this[*].id
}

output "internet_gateway_id" {
  description = "ID of the Internet Gateway"
  value       = aws_internet_gateway.this[0].id
}
```

### Module Composition

```hcl
# Root module using multiple modules
module "vpc" {
  source = "./modules/vpc"
  
  name               = "production"
  vpc_cidr           = "10.0.0.0/16"
  availability_zones = ["us-east-1a", "us-east-1b"]
  public_subnets     = ["10.0.1.0/24", "10.0.2.0/24"]
  private_subnets    = ["10.0.11.0/24", "10.0.12.0/24"]
  
  tags = {
    Environment = "production"
    ManagedBy   = "Terraform"
  }
}

module "eks" {
  source = "./modules/eks"
  
  cluster_name    = "production-cluster"
  vpc_id          = module.vpc.vpc_id
  subnet_ids      = module.vpc.private_subnet_ids
  cluster_version = "1.28"
  
  depends_on = [module.vpc]
}

module "rds" {
  source = "./modules/rds"
  
  identifier     = "production-db"
  vpc_id         = module.vpc.vpc_id
  subnet_ids     = module.vpc.private_subnet_ids
  engine_version = "15.4"
  
  depends_on = [module.vpc]
}
```

## 🔄 Workspaces

### Workspace Management

```bash
# Create workspaces
terraform workspace new development
terraform workspace new staging
terraform workspace new production

# List workspaces
terraform workspace list

# Select workspace
terraform workspace select production

# Show current workspace
terraform workspace show

# Delete workspace
terraform workspace delete development
```

### Workspace-Specific Configuration

```hcl
# Use workspace in configuration
locals {
  environment = terraform.workspace
  
  common_tags = {
    Environment = terraform.workspace
    ManagedBy   = "Terraform"
  }
  
  instance_counts = {
    development = 1
    staging     = 2
    production  = 5
  }
  
  instance_count = lookup(local.instance_counts, local.environment, 1)
}

resource "aws_instance" "web" {
  count = local.instance_count
  
  instance_type = var.instance_type
  
  tags = local.common_tags
}
```

### Workspace Variables

```hcl
# terraform.tfvars (default)
instance_type = "t2.micro"

# development.auto.tfvars
instance_type = "t2.micro"

# production.auto.tfvars
instance_type = "t3.medium"
```

## 🎨 Dynamic Blocks

### Dynamic Security Group Rules

```hcl
variable "ingress_rules" {
  description = "List of ingress rules"
  type = list(object({
    from_port   = number
    to_port     = number
    protocol    = string
    cidr_blocks = list(string)
    description = string
  }))
  default = []
}

resource "aws_security_group" "web" {
  name        = "web-sg"
  description = "Security group for web servers"
  
  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
      description = ingress.value.description
    }
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

### Dynamic Tags

```hcl
locals {
  common_tags = {
    Environment = var.environment
    ManagedBy   = "Terraform"
    Project     = var.project_name
  }
  
  additional_tags = {
    Team        = "DevOps"
    CostCenter  = "Engineering"
  }
  
  all_tags = merge(
    local.common_tags,
    local.additional_tags,
    var.extra_tags
  )
}

resource "aws_instance" "web" {
  tags = local.all_tags
}
```

## 🔤 For Expressions

### For Loops

```hcl
# Transform list
variable "subnets" {
  type = list(string)
  default = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
}

# Create map from list
locals {
  subnet_map = {
    for idx, subnet in var.subnets : "subnet-${idx + 1}" => subnet
  }
}

# Filter list
locals {
  public_subnets = [
    for subnet in var.subnets : subnet
    if can(regex("^10\\.0\\.[1-3]\\.", subnet))
  ]
}

# Transform map
locals {
  uppercase_tags = {
    for k, v in var.tags : upper(k) => upper(v)
  }
}
```

### For with Conditionals

```hcl
# Conditional creation
resource "aws_instance" "web" {
  for_each = {
    for k, v in var.instances : k => v
    if v.enabled == true
  }
  
  instance_type = each.value.instance_type
  # ...
}
```

## 🔐 Advanced State Management

### Remote State with S3

```hcl
terraform {
  backend "s3" {
    bucket         = "terraform-state-bucket"
    key            = "environments/production/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-state-lock"
    kms_key_id     = "arn:aws:kms:us-east-1:123456789012:key/abc123"
  }
}
```

### State Locking

```hcl
# DynamoDB table for state locking
resource "aws_dynamodb_table" "terraform_state_lock" {
  name           = "terraform-state-lock"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "LockID"
  
  attribute {
    name = "LockID"
    type = "S"
  }
  
  tags = {
    Name = "Terraform State Lock"
  }
}
```

### State Backend Migration

```bash
# Migrate from local to remote
terraform init -migrate-state

# Or manually
terraform init \
  -backend-config="bucket=my-bucket" \
  -backend-config="key=path/to/state" \
  -migrate-state
```

## 🧪 Testing

### Terraform Validate

```bash
# Validate syntax
terraform validate

# Validate with variables
terraform validate -var="instance_type=t3.medium"

# Validate in CI/CD
terraform init -backend=false
terraform validate
```

### Terratest

**Go-based testing framework**

```go
package test

import (
    "testing"
    "github.com/gruntwork-io/terratest/modules/terraform"
    "github.com/stretchr/testify/assert"
)

func TestTerraformAwsExample(t *testing.T) {
    terraformOptions := terraform.WithDefaultRetryableErrors(t, &terraform.Options{
        TerraformDir: "../examples/basic",
    })
    
    defer terraform.Destroy(t, terraformOptions)
    
    terraform.InitAndApply(t, terraformOptions)
    
    instanceId := terraform.Output(t, terraformOptions, "instance_id")
    assert.NotEmpty(t, instanceId)
}
```

### Checkov

**Static analysis**

```bash
# Install
pip install checkov

# Scan
checkov -d terraform/

# Scan with policy
checkov -d terraform/ --framework terraform --check CKV_AWS_20
```

## 🚀 Performance Optimization

### Parallelism

```bash
# Control parallel operations
terraform apply -parallelism=10

# Default is 10
```

### Targeted Operations

```bash
# Apply specific resources
terraform apply -target=aws_instance.web

# Multiple targets
terraform apply \
  -target=aws_instance.web \
  -target=aws_security_group.web
```

### Module Caching

```hcl
# Use module caching
terraform {
  # Cache modules
}
```

## 📊 Advanced Functions

### String Functions

```hcl
# String manipulation
upper("hello")                    # "HELLO"
lower("HELLO")                    # "hello"
title("hello world")              # "Hello World"
substr("hello", 0, 4)            # "hell"
replace("hello", "l", "L")       # "heLLo"
split(",", "a,b,c")               # ["a", "b", "c"]
join(",", ["a", "b", "c"])        # "a,b,c"
format("Hello, %s!", "World")     # "Hello, World!"
```

### Collection Functions

```hcl
# List functions
length(["a", "b", "c"])           # 3
contains(["a", "b"], "a")         # true
distinct(["a", "b", "a"])        # ["a", "b"]
reverse(["a", "b", "c"])         # ["c", "b", "a"]
sort(["c", "a", "b"])            # ["a", "b", "c"]

# Map functions
keys({a=1, b=2})                 # ["a", "b"]
values({a=1, b=2})                # [1, 2]
lookup({a=1, b=2}, "a", 0)       # 1
merge({a=1}, {b=2})               # {a=1, b=2}
```

### Type Conversion

```hcl
# Type conversion
tostring(123)                     # "123"
tonumber("123")                   # 123
tobool("true")                    # true
tolist({a=1, b=2})                # [1, 2]
tomap({a=1, b=2})                 # {a=1, b=2}
toset(["a", "b", "a"])            # ["a", "b"]
```

### Date/Time Functions

```hcl
# Date/time
timestamp()                       # Current timestamp
formatdate("YYYY-MM-DD", timestamp())  # "2024-01-15"
timeadd(timestamp(), "1h")        # Add 1 hour
```

### File Functions

```hcl
# File operations
file("path/to/file.txt")          # Read file
fileexists("path/to/file.txt")     # Check existence
templatefile("template.tpl", {    # Template file
  name = "World"
})
```

## 🎯 Advanced Patterns

### Conditional Resources

```hcl
# Create resource conditionally
resource "aws_instance" "web" {
  count = var.create_instance ? 1 : 0
  
  instance_type = var.instance_type
  # ...
}

# Multiple conditional resources
resource "aws_instance" "web" {
  for_each = var.create_instances ? var.instances : {}
  
  instance_type = each.value.instance_type
  # ...
}
```

### Data Source Dependencies

```hcl
# Use data sources for dynamic values
data "aws_ami" "latest" {
  most_recent = true
  owners      = ["amazon"]
  
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

resource "aws_instance" "web" {
  ami = data.aws_ami.latest.id
  # ...
}
```

### Null Resources

```hcl
# Execute local commands
resource "null_resource" "example" {
  triggers = {
    instance_id = aws_instance.web.id
  }
  
  provisioner "local-exec" {
    command = "echo Instance ${aws_instance.web.id} created"
  }
}
```

## ✅ Best Practices

### 1. Use Modules for Reusability

```hcl
# Create reusable modules
# Document modules well
# Version modules
```

### 2. Implement Proper State Management

```hcl
# Use remote state
# Enable state locking
# Backup state regularly
```

### 3. Use Workspaces for Environments

```hcl
# Separate workspaces per environment
# Use workspace-specific variables
# Don't mix environments
```

### 4. Validate and Test

```bash
# Always validate
terraform validate

# Test with plan
terraform plan

# Use testing frameworks
```

### 5. Document Everything

```hcl
# Document variables
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
}

# Document outputs
output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.web.id
}
```

## ✅ Mastery Checklist

- [ ] Create complex modules
- [ ] Use workspaces effectively
- [ ] Implement dynamic blocks
- [ ] Use for expressions
- [ ] Manage remote state
- [ ] Test Terraform code
- [ ] Optimize performance
- [ ] Use advanced functions
- [ ] Implement conditional resources
- [ ] Follow best practices

---

**Next Steps**:
- Learn [Terraform Basics](./terraform-basics.md) if needed
- Explore [AWS Core Services](./aws-core-services.md)
- Master [CI/CD](./ci-cd.md) integration

**Remember**: Advanced Terraform requires understanding of modules, state management, and testing. Start with basics, build complex modules gradually, and always test your infrastructure code. Good Terraform practices enable scalable, maintainable infrastructure.
