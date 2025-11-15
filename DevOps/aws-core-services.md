# AWS Core Services: Complete Guide for DevOps

## 🎯 Introduction

Amazon Web Services (AWS) is the leading cloud platform, providing over 200 services. This guide covers the essential AWS services every DevOps engineer must master for building, deploying, and managing cloud infrastructure.

### AWS Global Infrastructure

- **Regions**: 31+ geographic regions worldwide
- **Availability Zones**: 99+ availability zones
- **Edge Locations**: 400+ edge locations for CloudFront
- **Local Zones**: Low-latency compute closer to users

## 💻 Compute Services

### EC2 (Elastic Compute Cloud)

**Virtual servers in the cloud**

#### Instance Types

```bash
# General Purpose
t3.micro, t3.small, t3.medium    # Burstable performance
m5.large, m5.xlarge               # Balanced compute/memory/network

# Compute Optimized
c5.large, c5.xlarge               # High-performance processors

# Memory Optimized
r5.large, r5.xlarge               # High memory-to-vCPU ratio

# Storage Optimized
i3.large, i3.xlarge               # High IOPS SSD storage

# GPU Instances
p3.2xlarge, g4dn.xlarge           # GPU-accelerated computing
```

#### EC2 Best Practices

```hcl
# Terraform Example
resource "aws_instance" "web" {
  ami                    = data.aws_ami.amazon_linux.id
  instance_type          = "t3.medium"
  vpc_security_group_ids = [aws_security_group.web.id]
  subnet_id              = aws_subnet.public[0].id
  key_name               = aws_key_pair.main.key_name
  
  user_data = <<-EOF
    #!/bin/bash
    yum update -y
    yum install -y httpd
    systemctl start httpd
    systemctl enable httpd
  EOF
  
  tags = {
    Name        = "WebServer"
    Environment = "production"
    ManagedBy   = "Terraform"
  }
}
```

### Lambda (Serverless Functions)

**Run code without managing servers**

```python
# Lambda Function Example
import json
import boto3

def lambda_handler(event, context):
    s3 = boto3.client('s3')
    
    # Process event
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    # Your logic here
    response = s3.get_object(Bucket=bucket, Key=key)
    
    return {
        'statusCode': 200,
        'body': json.dumps('Success')
    }
```

**Lambda Configuration**:
- Memory: 128 MB to 10 GB
- Timeout: Up to 15 minutes
- Triggers: API Gateway, S3, DynamoDB, EventBridge, etc.
- Layers: Share code and dependencies

### ECS (Elastic Container Service)

**Fully managed container orchestration**

```json
{
  "family": "my-app",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "containerDefinitions": [
    {
      "name": "app",
      "image": "myapp:latest",
      "portMappings": [
        {
          "containerPort": 80,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {
          "name": "ENV",
          "value": "production"
        }
      ]
    }
  ]
}
```

### EKS (Elastic Kubernetes Service)

**Managed Kubernetes service**

```bash
# Create EKS cluster
aws eks create-cluster \
  --name my-cluster \
  --role-arn arn:aws:iam::123456789012:role/EKSClusterRole \
  --resources-vpc-config subnetIds=subnet-123,subnet-456

# Update kubeconfig
aws eks update-kubeconfig --name my-cluster --region us-east-1
```

## 📦 Storage Services

### S3 (Simple Storage Service)

**Object storage for any amount of data**

#### S3 Features

```hcl
# Terraform S3 Bucket
resource "aws_s3_bucket" "data" {
  bucket = "my-data-bucket-${random_id.bucket_suffix.hex}"
  
  tags = {
    Name        = "Data Bucket"
    Environment = "production"
  }
}

# Versioning
resource "aws_s3_bucket_versioning" "data" {
  bucket = aws_s3_bucket.data.id
  versioning_configuration {
    status = "Enabled"
  }
}

# Encryption
resource "aws_s3_bucket_server_side_encryption_configuration" "data" {
  bucket = aws_s3_bucket.data.id
  
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

# Lifecycle Policy
resource "aws_s3_bucket_lifecycle_configuration" "data" {
  bucket = aws_s3_bucket.data.id
  
  rule {
    id     = "transition-to-ia"
    status = "Enabled"
    
    transition {
      days          = 30
      storage_class = "STANDARD_IA"
    }
    
    transition {
      days          = 90
      storage_class = "GLACIER"
    }
  }
  
  rule {
    id     = "delete-old-versions"
    status = "Enabled"
    
    noncurrent_version_expiration {
      noncurrent_days = 90
    }
  }
}

# Public Access Block
resource "aws_s3_bucket_public_access_block" "data" {
  bucket = aws_s3_bucket.data.id
  
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets  = true
}
```

#### S3 Storage Classes

- **Standard**: General-purpose, frequently accessed
- **Standard-IA**: Infrequently accessed, lower cost
- **One Zone-IA**: Single AZ, lower cost
- **Glacier Instant Retrieval**: Archive with instant access
- **Glacier Flexible Retrieval**: Archive (expedited/standard/bulk)
- **Glacier Deep Archive**: Lowest cost, 12-hour retrieval
- **Intelligent-Tiering**: Automatic cost optimization

### EBS (Elastic Block Store)

**Block storage for EC2 instances**

```hcl
resource "aws_ebs_volume" "data" {
  availability_zone = aws_instance.web.availability_zone
  size              = 100
  type              = "gp3"
  encrypted         = true
  kms_key_id        = aws_kms_key.ebs.arn
  
  tags = {
    Name = "Data Volume"
  }
}

resource "aws_volume_attachment" "ebs_att" {
  device_name = "/dev/sdf"
  volume_id   = aws_ebs_volume.data.id
  instance_id = aws_instance.web.id
}
```

**EBS Volume Types**:
- **gp3**: General Purpose SSD (latest, best price/performance)
- **gp2**: General Purpose SSD (legacy)
- **io1/io2**: Provisioned IOPS SSD
- **st1**: Throughput Optimized HDD
- **sc1**: Cold HDD

### EFS (Elastic File System)

**Managed NFS for EC2**

```hcl
resource "aws_efs_file_system" "shared" {
  creation_token = "shared-storage"
  encrypted      = true
  kms_key_id     = aws_kms_key.efs.arn
  
  lifecycle_policy {
    transition_to_ia = "AFTER_30_DAYS"
  }
  
  tags = {
    Name = "Shared Storage"
  }
}

resource "aws_efs_mount_target" "shared" {
  file_system_id  = aws_efs_file_system.shared.id
  subnet_id       = aws_subnet.private[0].id
  security_groups = [aws_security_group.efs.id]
}
```

## 🌐 Networking Services

### VPC (Virtual Private Cloud)

**Isolated network environment**

```hcl
# VPC
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = {
    Name = "Main VPC"
  }
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = {
    Name = "Main IGW"
  }
}

# Public Subnets
resource "aws_subnet" "public" {
  count             = 2
  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet(aws_vpc.main.cidr_block, 8, count.index)
  availability_zone = data.aws_availability_zones.available.names[count.index]
  map_public_ip_on_launch = true
  
  tags = {
    Name = "Public Subnet ${count.index + 1}"
    Type = "Public"
  }
}

# Private Subnets
resource "aws_subnet" "private" {
  count             = 2
  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet(aws_vpc.main.cidr_block, 8, count.index + 10)
  availability_zone = data.aws_availability_zones.available.names[count.index]
  
  tags = {
    Name = "Private Subnet ${count.index + 1}"
    Type = "Private"
  }
}

# NAT Gateway
resource "aws_eip" "nat" {
  domain = "vpc"
  
  tags = {
    Name = "NAT Gateway EIP"
  }
}

resource "aws_nat_gateway" "main" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public[0].id
  
  tags = {
    Name = "Main NAT Gateway"
  }
}

# Route Tables
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }
  
  tags = {
    Name = "Public Route Table"
  }
}

resource "aws_route_table" "private" {
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.main.id
  }
  
  tags = {
    Name = "Private Route Table"
  }
}
```

### CloudFront (CDN)

**Global content delivery network**

```hcl
resource "aws_cloudfront_distribution" "cdn" {
  origin {
    domain_name = aws_s3_bucket.website.bucket_regional_domain_name
    origin_id   = "S3-${aws_s3_bucket.website.id}"
    
    s3_origin_config {
      origin_access_identity = aws_cloudfront_origin_access_identity.oai.cloudfront_access_identity_path
    }
  }
  
  enabled             = true
  is_ipv6_enabled     = true
  default_root_object = "index.html"
  
  default_cache_behavior {
    allowed_methods  = ["DELETE", "GET", "HEAD", "OPTIONS", "PATCH", "POST", "PUT"]
    cached_methods   = ["GET", "HEAD"]
    target_origin_id = "S3-${aws_s3_bucket.website.id}"
    
    forwarded_values {
      query_string = false
      cookies {
        forward = "none"
      }
    }
    
    viewer_protocol_policy = "redirect-to-https"
    min_ttl                = 0
    default_ttl            = 3600
    max_ttl                = 86400
  }
  
  restrictions {
    geo_restriction {
      restriction_type = "whitelist"
      locations        = ["US", "CA", "GB", "DE"]
    }
  }
  
  viewer_certificate {
    cloudfront_default_certificate = true
  }
}
```

### Route 53 (DNS)

**Scalable DNS and domain name registration**

```hcl
# Hosted Zone
resource "aws_route53_zone" "main" {
  name = "example.com"
  
  tags = {
    Environment = "production"
  }
}

# A Record
resource "aws_route53_record" "www" {
  zone_id = aws_route53_zone.main.zone_id
  name    = "www.example.com"
  type    = "A"
  
  alias {
    name                   = aws_cloudfront_distribution.cdn.domain_name
    zone_id                = aws_cloudfront_distribution.cdn.hosted_zone_id
    evaluate_target_health = false
  }
}

# CNAME Record
resource "aws_route53_record" "api" {
  zone_id = aws_route53_zone.main.zone_id
  name    = "api.example.com"
  type    = "CNAME"
  ttl     = 300
  records = ["api-lb-123456789.us-east-1.elb.amazonaws.com"]
}
```

### Load Balancer

**Application Load Balancer (ALB)**

```hcl
resource "aws_lb" "main" {
  name               = "main-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb.id]
  subnets            = aws_subnet.public[*].id
  
  enable_deletion_protection = false
  enable_http2              = true
  enable_cross_zone_load_balancing = true
  
  tags = {
    Environment = "production"
  }
}

resource "aws_lb_target_group" "app" {
  name     = "app-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.main.id
  
  health_check {
    enabled             = true
    healthy_threshold   = 2
    unhealthy_threshold = 2
    timeout             = 5
    interval            = 30
    path                = "/health"
    matcher             = "200"
  }
  
  stickiness {
    type            = "lb_cookie"
    cookie_duration = 86400
  }
}

resource "aws_lb_listener" "app" {
  load_balancer_arn = aws_lb.main.arn
  port              = "80"
  protocol          = "HTTP"
  
  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.app.arn
  }
}
```

## 🗄️ Database Services

### RDS (Relational Database Service)

**Managed relational databases**

```hcl
# RDS Instance
resource "aws_db_instance" "postgres" {
  identifier     = "postgres-db"
  engine         = "postgres"
  engine_version = "15.4"
  instance_class = "db.t3.medium"
  
  allocated_storage     = 100
  max_allocated_storage = 200
  storage_type          = "gp3"
  storage_encrypted      = true
  
  db_name  = "mydb"
  username = "admin"
  password = var.db_password
  
  vpc_security_group_ids = [aws_security_group.rds.id]
  db_subnet_group_name   = aws_db_subnet_group.main.name
  
  backup_retention_period = 7
  backup_window          = "03:00-04:00"
  maintenance_window     = "mon:04:00-mon:05:00"
  
  enabled_cloudwatch_logs_exports = ["postgresql", "upgrade"]
  
  performance_insights_enabled = true
  monitoring_interval         = 60
  monitoring_role_arn         = aws_iam_role.rds_monitoring.arn
  
  skip_final_snapshot = false
  final_snapshot_identifier = "postgres-final-snapshot-${formatdate("YYYY-MM-DD-hhmm", timestamp())}"
  
  tags = {
    Name = "PostgreSQL Database"
  }
}

# DB Subnet Group
resource "aws_db_subnet_group" "main" {
  name       = "main-db-subnet-group"
  subnet_ids = aws_subnet.private[*].id
  
  tags = {
    Name = "DB Subnet Group"
  }
}
```

### DynamoDB (NoSQL Database)

**Fully managed NoSQL database**

```hcl
resource "aws_dynamodb_table" "users" {
  name           = "users"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "user_id"
  
  attribute {
    name = "user_id"
    type = "S"
  }
  
  attribute {
    name = "email"
    type = "S"
  }
  
  global_secondary_index {
    name     = "email-index"
    hash_key = "email"
  }
  
  point_in_time_recovery {
    enabled = true
  }
  
  server_side_encryption {
    enabled = true
  }
  
  tags = {
    Name        = "Users Table"
    Environment = "production"
  }
}
```

### ElastiCache (In-Memory Cache)

**Redis and Memcached**

```hcl
# Redis Cluster
resource "aws_elasticache_replication_group" "redis" {
  replication_group_id       = "redis-cluster"
  description                = "Redis cluster for caching"
  
  engine                     = "redis"
  engine_version             = "7.0"
  node_type                  = "cache.t3.micro"
  port                       = 6379
  parameter_group_name       = "default.redis7"
  
  num_cache_clusters         = 2
  
  subnet_group_name          = aws_elasticache_subnet_group.main.name
  security_group_ids         = [aws_security_group.redis.id]
  
  at_rest_encryption_enabled = true
  transit_encryption_enabled = true
  auth_token                 = var.redis_auth_token
  
  automatic_failover_enabled  = true
  multi_az_enabled           = true
  
  snapshot_retention_limit    = 5
  snapshot_window            = "03:00-05:00"
  
  tags = {
    Name = "Redis Cluster"
  }
}
```

## 🔐 Security Services

### IAM (Identity and Access Management)

**User and access management**

```hcl
# IAM Role for EC2
resource "aws_iam_role" "ec2_role" {
  name = "ec2-role"
  
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "ec2.amazonaws.com"
        }
      }
    ]
  })
  
  tags = {
    Name = "EC2 Role"
  }
}

# IAM Policy
resource "aws_iam_policy" "s3_access" {
  name        = "s3-access-policy"
  description = "Policy for S3 access"
  
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "s3:GetObject",
          "s3:PutObject"
        ]
        Resource = "${aws_s3_bucket.data.arn}/*"
      }
    ]
  })
}

# Attach Policy to Role
resource "aws_iam_role_policy_attachment" "ec2_s3" {
  role       = aws_iam_role.ec2_role.name
  policy_arn = aws_iam_policy.s3_access.arn
}

# Instance Profile
resource "aws_iam_instance_profile" "ec2" {
  name = "ec2-instance-profile"
  role = aws_iam_role.ec2_role.name
}
```

### Secrets Manager

**Secrets storage and rotation**

```hcl
resource "aws_secretsmanager_secret" "db_password" {
  name                    = "db-password"
  description             = "Database password"
  recovery_window_in_days = 7
  
  tags = {
    Name = "DB Password"
  }
}

resource "aws_secretsmanager_secret_version" "db_password" {
  secret_id = aws_secretsmanager_secret.db_password.id
  secret_string = jsonencode({
    username = "admin"
    password = var.db_password
  })
}
```

### KMS (Key Management Service)

**Encryption key management**

```hcl
resource "aws_kms_key" "ebs" {
  description             = "KMS key for EBS encryption"
  deletion_window_in_days = 10
  enable_key_rotation     = true
  
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid    = "Enable IAM User Permissions"
        Effect = "Allow"
        Principal = {
          AWS = "arn:aws:iam::${data.aws_caller_identity.current.account_id}:root"
        }
        Action   = "kms:*"
        Resource = "*"
      }
    ]
  })
  
  tags = {
    Name = "EBS Encryption Key"
  }
}

resource "aws_kms_alias" "ebs" {
  name          = "alias/ebs-encryption"
  target_key_id = aws_kms_key.ebs.key_id
}
```

## 📊 Monitoring Services

### CloudWatch

**Monitoring and observability**

```hcl
# CloudWatch Log Group
resource "aws_cloudwatch_log_group" "app" {
  name              = "/aws/ec2/myapp"
  retention_in_days = 30
  
  tags = {
    Name = "Application Logs"
  }
}

# CloudWatch Alarm
resource "aws_cloudwatch_metric_alarm" "cpu" {
  alarm_name          = "high-cpu-utilization"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 2
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = 300
  statistic           = "Average"
  threshold           = 80
  alarm_description   = "This metric monitors ec2 cpu utilization"
  
  dimensions = {
    InstanceId = aws_instance.web.id
  }
  
  alarm_actions = [aws_sns_topic.alerts.arn]
}

# CloudWatch Dashboard
resource "aws_cloudwatch_dashboard" "main" {
  dashboard_name = "my-dashboard"
  
  dashboard_body = jsonencode({
    widgets = [
      {
        type   = "metric"
        x      = 0
        y      = 0
        width  = 12
        height = 6
        
        properties = {
          metrics = [
            ["AWS/EC2", "CPUUtilization", "InstanceId", aws_instance.web.id]
          ]
          period = 300
          stat   = "Average"
          region = "us-east-1"
          title  = "EC2 CPU Utilization"
        }
      }
    ]
  })
}
```

### CloudTrail

**API logging and auditing**

```hcl
resource "aws_cloudtrail" "main" {
  name                          = "main-trail"
  s3_bucket_name                = aws_s3_bucket.cloudtrail.id
  include_global_service_events = true
  is_multi_region_trail         = true
  enable_logging                = true
  
  event_selector {
    read_write_type                 = "All"
    include_management_events       = true
    
    data_resource {
      type   = "AWS::S3::Object"
      values = ["${aws_s3_bucket.data.arn}/*"]
    }
  }
  
  tags = {
    Name = "Main CloudTrail"
  }
}
```

## 🎯 Best Practices

### 1. Use IAM Roles, Not Keys

```hcl
# Good - Use IAM roles
resource "aws_iam_role" "ec2_role" {
  # ...
}

# Bad - Don't hardcode keys
# AWS_ACCESS_KEY_ID = "AKIA..."
```

### 2. Enable Encryption

```hcl
# Encrypt everything
storage_encrypted = true
encrypted         = true
```

### 3. Use VPC for Isolation

```hcl
# Private subnets for databases
# Public subnets for load balancers
```

### 4. Implement Tagging

```hcl
tags = {
  Name        = "Resource Name"
  Environment = "production"
  ManagedBy   = "Terraform"
  Project     = "MyProject"
}
```

### 5. Enable Monitoring

```hcl
# CloudWatch alarms
# CloudTrail logging
# Performance Insights
```

### 6. Cost Optimization

- Use Reserved Instances for predictable workloads
- Use Spot Instances for flexible workloads
- Right-size instances
- Use S3 lifecycle policies
- Enable auto-scaling

## ✅ Mastery Checklist

- [ ] Understand EC2 instance types
- [ ] Configure VPC and networking
- [ ] Set up S3 with proper policies
- [ ] Deploy Lambda functions
- [ ] Use RDS for databases
- [ ] Configure IAM properly
- [ ] Set up CloudWatch monitoring
- [ ] Implement security best practices
- [ ] Use Terraform for infrastructure
- [ ] Optimize costs

---

**Next Steps**:
- Learn [Terraform](./terraform-basics.md) for Infrastructure as Code
- Explore [Kubernetes](./kubernetes-intro.md) for container orchestration
- Master [CI/CD](./ci-cd.md) for automation

**Remember**: AWS is vast. Start with core services (EC2, S3, VPC, IAM), understand networking, implement security from the start, and always monitor your resources. Use Infrastructure as Code for everything.
