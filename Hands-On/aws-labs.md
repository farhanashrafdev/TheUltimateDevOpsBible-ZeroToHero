# AWS Hands-On Labs

## 🎯 Overview

Deploy and manage infrastructure on AWS using Terraform and the AWS CLI.

## 📚 Lab 1: VPC with Terraform

**Objective**: Create a production-ready VPC

### Terraform Configuration

```hcl
# main.tf
provider "aws" {
  region = "us-east-1"
}

module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
  
  name = "lab-vpc"
  cidr = "10.0.0.0/16"
  
  azs             = ["us-east-1a", "us-east-1b"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24"]
  
  enable_nat_gateway = true
  single_nat_gateway = true
  
  tags = {
    Environment = "lab"
    Terraform   = "true"
  }
}
```

### Deploy

```bash
terraform init
terraform plan
terraform apply
```

---

## 📚 Lab 2: EKS Cluster

**Objective**: Deploy Kubernetes on AWS

```hcl
module "eks" {
  source = "terraform-aws-modules/eks/aws"
  
  cluster_name    = "lab-eks"
  cluster_version = "1.28"
  
  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets
  
  eks_managed_node_groups = {
    default = {
      instance_types = ["t3.medium"]
      min_size       = 1
      max_size       = 3
      desired_size   = 2
    }
  }
}
```

### Connect

```bash
aws eks update-kubeconfig --name lab-eks --region us-east-1
kubectl get nodes
```

---

## 📚 Lab 3: EC2 Auto Scaling

**Objective**: Create auto-scaling web tier

```hcl
resource "aws_launch_template" "web" {
  name_prefix   = "web-"
  image_id      = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"
  
  user_data = base64encode(<<-EOF
    #!/bin/bash
    yum install -y httpd
    systemctl start httpd
    echo "Hello from $(hostname)" > /var/www/html/index.html
  EOF
  )
}

resource "aws_autoscaling_group" "web" {
  desired_capacity    = 2
  max_size            = 4
  min_size            = 1
  vpc_zone_identifier = module.vpc.private_subnets
  
  launch_template {
    id      = aws_launch_template.web.id
    version = "$Latest"
  }
}
```

---

## ✅ Cleanup

```bash
terraform destroy
```

