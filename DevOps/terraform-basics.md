# Terraform: Complete Infrastructure as Code Guide

## 🎯 Introduction

Terraform is an open-source Infrastructure as Code (IaC) tool created by HashiCorp. It enables you to define, provision, and manage cloud infrastructure using declarative configuration files. Terraform supports multiple cloud providers and services.

### What is Infrastructure as Code?

**Infrastructure as Code (IaC)** is the practice of managing and provisioning infrastructure through machine-readable definition files, rather than through manual processes.

**Benefits**:
- **Version Control**: Track infrastructure changes
- **Reproducibility**: Create identical environments
- **Consistency**: Eliminate configuration drift
- **Documentation**: Code serves as documentation
- **Collaboration**: Team can review and contribute
- **Automation**: Reduce manual errors
- **Testing**: Test infrastructure changes

### Why Terraform?

1. **Multi-Cloud**: Works with AWS, Azure, GCP, and 100+ providers
2. **Declarative**: Describe desired state, Terraform figures out how
3. **State Management**: Tracks infrastructure state
4. **Plan Before Apply**: Preview changes before execution
5. **Modular**: Reusable modules for common patterns
6. **Large Ecosystem**: Extensive provider and module ecosystem
7. **Mature**: Battle-tested in production

### Terraform vs Alternatives

**Terraform vs CloudFormation**:
- Terraform: Multi-cloud, HCL syntax
- CloudFormation: AWS-only, JSON/YAML

**Terraform vs Ansible**:
- Terraform: Infrastructure provisioning, declarative
- Ansible: Configuration management, procedural

**Terraform vs Pulumi**:
- Terraform: HCL syntax, mature
- Pulumi: General-purpose languages, newer

## 📚 Core Concepts

### 1. Providers

**Providers** are plugins that Terraform uses to interact with cloud platforms and services.

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
    google = {
      source  = "hashicorp/google"
      version = "~> 4.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
  
  default_tags {
    tags = {
      Environment = "production"
      ManagedBy   = "Terraform"
    }
  }
}
```

### 2. Resources

**Resources** are the most important element in Terraform. They represent infrastructure objects.

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  tags = {
    Name = "WebServer"
  }
}
```

**Resource Syntax**:
```
resource "PROVIDER_TYPE" "RESOURCE_NAME" {
  # Configuration
}
```

### 3. Data Sources

**Data Sources** allow you to fetch information about existing infrastructure.

```hcl
data "aws_ami" "latest_ubuntu" {
  most_recent = true
  owners      = ["099720109477"] # Canonical
  
  filter {
    name   = "name"
    values = ["ubuntu/images/h2-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
  
  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

resource "aws_instance" "web" {
  ami           = data.aws_ami.latest_ubuntu.id
  instance_type = "t2.micro"
}
```

### 4. Variables

**Variables** make configurations reusable and flexible.

```hcl
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
  
  validation {
    condition     = can(regex("^t[23]\\..*", var.instance_type))
    error_message = "Instance type must be t2 or t3 family."
  }
}

variable "environment" {
  description = "Environment name"
  type        = string
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "tags" {
  description = "Resource tags"
  type        = map(string)
  default     = {}
}
```

**Using Variables**:
```hcl
resource "aws_instance" "web" {
  instance_type = var.instance_type
  
  tags = merge(
    var.tags,
    {
      Environment = var.environment
    }
  )
}
```

**Variable Files**:
```hcl
# terraform.tfvars
instance_type = "t3.medium"
environment   = "production"
tags = {
  Project = "MyProject"
  Team    = "DevOps"
}
```

### 5. Outputs

**Outputs** expose information about created resources.

```hcl
output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.web.id
}

output "instance_public_ip" {
  description = "Public IP address of the EC2 instance"
  value       = aws_instance.web.public_ip
}

output "instance_arn" {
  description = "ARN of the EC2 instance"
  value       = aws_instance.web.arn
  sensitive   = false
}
```

### 6. State

**State** is Terraform's record of what infrastructure it manages.

**Local State**:
```hcl
terraform {
  backend "local" {
    path = "terraform.tfstate"
  }
}
```

**Remote State (S3)**:
```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "environments/production/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-state-lock"
  }
}
```

## 🔧 Essential Commands

### Initialization

```bash
# Initialize Terraform
terraform init

# Initialize with backend migration
terraform init -migrate-state

# Initialize with backend configuration
terraform init -backend-config="bucket=my-bucket"

# Upgrade providers
terraform init -upgrade
```

### Planning

```bash
# Create execution plan
terraform plan

# Save plan to file
terraform plan -out=tfplan

# Use saved plan
terraform apply tfplan

# Plan with variables
terraform plan -var="instance_type=t3.large"

# Plan with variable file
terraform plan -var-file="production.tfvars"

# Plan for specific resource
terraform plan -target=aws_instance.web

# Refresh state before plan
terraform plan -refresh=true
```

### Applying

```bash
# Apply changes
terraform apply

# Apply with auto-approve
terraform apply -auto-approve

# Apply saved plan
terraform apply tfplan

# Apply with parallelism
terraform apply -parallelism=10
```

### Destroying

```bash
# Destroy infrastructure
terraform destroy

# Destroy specific resource
terraform destroy -target=aws_instance.web

# Destroy with auto-approve
terraform destroy -auto-approve
```

### State Management

```bash
# List resources in state
terraform state list

# Show resource details
terraform state show aws_instance.web

# Move resource in state
terraform state mv aws_instance.old aws_instance.new

# Remove resource from state
terraform state rm aws_instance.web

# Pull state
terraform state pull > terraform.tfstate

# Push state
terraform state push terraform.tfstate
```

### Importing

```bash
# Import existing resource
terraform import aws_instance.web i-1234567890abcdef0

# Import with resource address
terraform import 'aws_instance.web[0]' i-1234567890abcdef0
```

### Validation and Formatting

```bash
# Validate configuration
terraform validate

# Format code
terraform fmt

# Format recursively
terraform fmt -recursive

# Check formatting
terraform fmt -check
```

### Workspaces

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

## 📝 Complete Examples

### Example 1: AWS EC2 Instance

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

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}

variable "key_name" {
  description = "SSH key pair name"
  type        = string
}

data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]
  
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

resource "aws_security_group" "web" {
  name        = "web-sg"
  description = "Security group for web server"
  
  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  ingress {
    description = "HTTPS"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  ingress {
    description = "SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  tags = {
    Name = "web-sg"
  }
}

resource "aws_instance" "web" {
  ami                    = data.aws_ami.amazon_linux.id
  instance_type          = var.instance_type
  key_name               = var.key_name
  vpc_security_group_ids = [aws_security_group.web.id]
  
  user_data = <<-EOF
    #!/bin/bash
    yum update -y
    yum install -y httpd
    systemctl start httpd
    systemctl enable httpd
    echo "<h1>Hello from Terraform</h1>" > /var/www/html/index.html
  EOF
  
  tags = {
    Name = "web-server"
  }
}

output "instance_id" {
  value = aws_instance.web.id
}

output "instance_public_ip" {
  value = aws_instance.web.public_ip
}

output "instance_public_dns" {
  value = aws_instance.web.public_dns
}
```

### Example 2: VPC with Subnets

```hcl
variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "availability_zones" {
  description = "Availability zones"
  type        = list(string)
  default     = ["us-east-1a", "us-east-1b"]
}

resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = {
    Name = "main-vpc"
  }
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = {
    Name = "main-igw"
  }
}

resource "aws_subnet" "public" {
  count             = length(var.availability_zones)
  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet(var.vpc_cidr, 8, count.index)
  availability_zone = var.availability_zones[count.index]
  
  map_public_ip_on_launch = true
  
  tags = {
    Name = "public-subnet-${count.index + 1}"
  }
}

resource "aws_subnet" "private" {
  count             = length(var.availability_zones)
  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet(var.vpc_cidr, 8, count.index + 10)
  availability_zone = var.availability_zones[count.index]
  
  tags = {
    Name = "private-subnet-${count.index + 1}"
  }
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }
  
  tags = {
    Name = "public-rt"
  }
}

resource "aws_route_table_association" "public" {
  count          = length(aws_subnet.public)
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

output "vpc_id" {
  value = aws_vpc.main.id
}

output "public_subnet_ids" {
  value = aws_subnet.public[*].id
}

output "private_subnet_ids" {
  value = aws_subnet.private[*].id
}
```

### Example 3: S3 Bucket with Versioning

```hcl
variable "bucket_name" {
  description = "S3 bucket name"
  type        = string
}

resource "aws_s3_bucket" "main" {
  bucket = var.bucket_name
  
  tags = {
    Name        = var.bucket_name
    Environment = "production"
  }
}

resource "aws_s3_bucket_versioning" "main" {
  bucket = aws_s3_bucket.main.id
  
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "main" {
  bucket = aws_s3_bucket.main.id
  
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

resource "aws_s3_bucket_public_access_block" "main" {
  bucket = aws_s3_bucket.main.id
  
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_lifecycle_configuration" "main" {
  bucket = aws_s3_bucket.main.id
  
  rule {
    id     = "delete-old-versions"
    status = "Enabled"
    
    noncurrent_version_expiration {
      noncurrent_days = 90
    }
  }
  
  rule {
    id     = "transition-to-ia"
    status = "Enabled"
    
    transition {
      days          = 30
      storage_class = "STANDARD_IA"
    }
  }
  
  rule {
    id     = "transition-to-glacier"
    status = "Enabled"
    
    transition {
      days          = 90
      storage_class = "GLACIER"
    }
  }
}
```

## 🏗️ Modules

**Modules** are containers for multiple resources that are used together.

### Using Modules

```hcl
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"
  
  name = "my-vpc"
  cidr = "10.0.0.0/16"
  
  azs             = ["us-east-1a", "us-east-1b"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24"]
  
  enable_nat_gateway = true
  enable_vpn_gateway = false
  
  tags = {
    Terraform   = "true"
    Environment = "production"
  }
}
```

### Creating Modules

```
modules/
  ec2/
    main.tf
    variables.tf
    outputs.tf
    README.md
```

**modules/ec2/main.tf**:
```hcl
resource "aws_instance" "this" {
  ami           = var.ami
  instance_type = var.instance_type
  
  tags = var.tags
}
```

**modules/ec2/variables.tf**:
```hcl
variable "ami" {
  description = "AMI ID"
  type        = string
}

variable "instance_type" {
  description = "Instance type"
  type        = string
  default     = "t2.micro"
}

variable "tags" {
  description = "Resource tags"
  type        = map(string)
  default     = {}
}
```

**modules/ec2/outputs.tf**:
```hcl
output "instance_id" {
  value = aws_instance.this.id
}

output "instance_arn" {
  value = aws_instance.this.arn
}
```

**Using the module**:
```hcl
module "web_server" {
  source = "./modules/ec2"
  
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.medium"
  
  tags = {
    Name = "web-server"
  }
}

output "web_server_id" {
  value = module.web_server.instance_id
}
```

## 🔐 Best Practices

### 1. Version Control Everything

- Store all Terraform files in Git
- Use meaningful commit messages
- Tag releases
- Review changes via pull requests

### 2. Use Remote State

```hcl
terraform {
  backend "s3" {
    bucket         = "terraform-state-bucket"
    key            = "environments/production/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-state-lock"
  }
}
```

### 3. Use Workspaces for Environments

```bash
terraform workspace new production
terraform workspace new staging
terraform workspace new development
```

### 4. Validate and Format

```bash
# Add to CI/CD pipeline
terraform fmt -check
terraform validate
terraform plan
```

### 5. Use Variables, Not Hardcoded Values

```hcl
# Bad
resource "aws_instance" "web" {
  instance_type = "t2.micro"
}

# Good
variable "instance_type" {
  type    = string
  default = "t2.micro"
}

resource "aws_instance" "web" {
  instance_type = var.instance_type
}
```

### 6. Tag Resources

```hcl
locals {
  common_tags = {
    Environment = var.environment
    ManagedBy   = "Terraform"
    Project     = var.project_name
  }
}

resource "aws_instance" "web" {
  tags = merge(local.common_tags, {
    Name = "web-server"
  })
}
```

### 7. Use Data Sources

```hcl
# Instead of hardcoding AMI IDs
data "aws_ami" "latest" {
  most_recent = true
  owners      = ["amazon"]
  
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}
```

### 8. Plan Before Apply

Always review the plan before applying:
```bash
terraform plan -out=tfplan
terraform apply tfplan
```

### 9. Use Modules for Reusability

- Create modules for common patterns
- Use community modules
- Document modules well

### 10. Secure Sensitive Data

```hcl
# Use sensitive variables
variable "db_password" {
  type      = string
  sensitive = true
}

# Use secrets management
data "aws_secretsmanager_secret_version" "db" {
  secret_id = "db-credentials"
}

locals {
  db_password = jsondecode(data.aws_secretsmanager_secret_version.db.secret_string)["password"]
}
```

## 🔍 Troubleshooting

### Common Issues

**1. State Lock**
```bash
# If state is locked
terraform force-unlock <LOCK_ID>
```

**2. State Drift**
```bash
# Refresh state
terraform refresh

# Plan to see differences
terraform plan
```

**3. Import Errors**
```bash
# Import with correct resource address
terraform import 'aws_instance.web[0]' i-1234567890abcdef0
```

**4. Provider Errors**
```bash
# Update providers
terraform init -upgrade

# Clear provider cache
rm -rf .terraform
terraform init
```

## ✅ Mastery Checklist

- [ ] Understand Terraform concepts
- [ ] Write basic configurations
- [ ] Use variables and outputs
- [ ] Manage state properly
- [ ] Create and use modules
- [ ] Use remote state backends
- [ ] Implement best practices
- [ ] Troubleshoot common issues
- [ ] Use workspaces
- [ ] Integrate with CI/CD

---

**Next Steps**:
- Learn [Advanced Terraform](./terraform-advanced.md) patterns
- Explore Terraform Cloud/Enterprise
- Study provider-specific resources
- Practice with real projects

**Remember**: Terraform is powerful but requires careful state management. Always plan before apply, use version control, and follow best practices. Start simple and gradually build complexity.
