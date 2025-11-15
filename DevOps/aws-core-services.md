# AWS Core Services

## 🎯 Essential AWS Services

### Compute
- **EC2**: Virtual servers
- **Lambda**: Serverless functions
- **ECS/EKS**: Container services
- **Elastic Beanstalk**: Platform as a Service

### Storage
- **S3**: Object storage
- **EBS**: Block storage
- **EFS**: File storage
- **Glacier**: Archive storage

### Networking
- **VPC**: Virtual Private Cloud
- **CloudFront**: CDN
- **Route 53**: DNS
- **API Gateway**: API management
- **Load Balancer**: Traffic distribution

### Database
- **RDS**: Managed relational databases
- **DynamoDB**: NoSQL database
- **ElastiCache**: In-memory cache
- **Redshift**: Data warehouse

### Security
- **IAM**: Identity and Access Management
- **Secrets Manager**: Secrets storage
- **KMS**: Key management
- **WAF**: Web Application Firewall

### Monitoring
- **CloudWatch**: Monitoring and logging
- **CloudTrail**: Audit logging
- **X-Ray**: Distributed tracing

## 📝 Common Patterns

### EC2 Instance
```hcl
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  vpc_security_group_ids = [aws_security_group.web.id]
  
  tags = {
    Name = "WebServer"
  }
}
```

### S3 Bucket
```hcl
resource "aws_s3_bucket" "data" {
  bucket = "my-data-bucket"
  
  versioning {
    enabled = true
  }
}
```

### VPC
```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  
  tags = {
    Name = "Main VPC"
  }
}
```

## ✅ Best Practices

- Use IAM roles, not keys
- Enable CloudTrail
- Use VPC for isolation
- Implement proper tagging
- Use CloudWatch for monitoring
- Follow security best practices
- Cost optimization

---

**Next**: Learn Kubernetes introduction.

