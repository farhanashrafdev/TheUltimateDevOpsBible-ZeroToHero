# Terraform Basics

## 🎯 Introduction

Terraform is an Infrastructure as Code (IaC) tool that enables you to define and provision infrastructure using declarative configuration files.

## 📚 Core Concepts

### Key Components
- **Providers**: Plugins for cloud platforms
- **Resources**: Infrastructure components
- **State**: Current infrastructure state
- **Modules**: Reusable configurations

## 📝 Basic Example

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  tags = {
    Name = "WebServer"
  }
}
```

## 🔧 Core Commands

```bash
# Initialize
terraform init

# Plan changes
terraform plan

# Apply changes
terraform apply

# Destroy infrastructure
terraform destroy

# Show state
terraform show

# Format code
terraform fmt

# Validate
terraform validate
```

## 📦 State Management

### Local State
```hcl
terraform {
  backend "local" {
    path = "terraform.tfstate"
  }
}
```

### Remote State (S3)
```hcl
terraform {
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "path/to/state"
    region = "us-east-1"
  }
}
```

## ✅ Best Practices

- Version control all code
- Use remote state
- Implement modules
- Use variables
- Tag resources
- Plan before apply
- Use workspaces

---

**Next**: Learn advanced Terraform patterns and modules.

