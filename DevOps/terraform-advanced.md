# Terraform Advanced

## 🎯 Advanced Patterns

### Modules
```hcl
module "vpc" {
  source = "./modules/vpc"
  
  vpc_cidr = "10.0.0.0/16"
  environment = "production"
}
```

### Workspaces
```bash
terraform workspace new production
terraform workspace select production
```

### Data Sources
```hcl
data "aws_ami" "latest" {
  most_recent = true
  owners      = ["amazon"]
  
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*"]
  }
}
```

### Outputs
```hcl
output "instance_ip" {
  value = aws_instance.web.public_ip
  description = "Public IP of the instance"
}
```

### Variables
```hcl
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}
```

## 🔧 Advanced Features

### Dynamic Blocks
```hcl
dynamic "ingress" {
  for_each = var.ports
  content {
    from_port   = ingress.value
    to_port     = ingress.value
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

### Conditional Resources
```hcl
resource "aws_instance" "web" {
  count = var.create_instance ? 1 : 0
  # ...
}
```

### For Expressions
```hcl
tags = {
  for k, v in var.tags : k => v
}
```

## ✅ Best Practices

- Use modules for reusability
- Implement proper state management
- Use workspaces for environments
- Validate with terraform validate
- Use remote state backends
- Implement CI/CD for Terraform
- Document all resources

---

**Next**: Learn AWS core services.

