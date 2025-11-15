# AWS Labs

## 🎯 AWS Practical Exercises

### Lab 1: EC2 Basics
**Objective**: Launch and manage EC2 instances

**Tasks**:
1. Launch EC2 instance
2. Connect via SSH
3. Configure security groups
4. Create AMI

**Commands**:
```bash
aws ec2 run-instances --image-id ami-xxx --instance-type t2.micro
aws ec2 describe-instances
aws ec2 create-image --instance-id i-xxx --name my-ami
```

### Lab 2: S3 Operations
**Objective**: Work with S3

**Tasks**:
1. Create bucket
2. Upload files
3. Set permissions
4. Enable versioning

**Commands**:
```bash
aws s3 mb s3://my-bucket
aws s3 cp file.txt s3://my-bucket/
aws s3 ls s3://my-bucket
```

### Lab 3: VPC Setup
**Objective**: Configure VPC

**Tasks**:
1. Create VPC
2. Create subnets
3. Configure route tables
4. Set up internet gateway

**Terraform Example**:
```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}
```

### Lab 4: RDS Database
**Objective**: Create managed database

**Tasks**:
1. Create RDS instance
2. Configure security
3. Connect to database
4. Create backups

**Commands**:
```bash
aws rds create-db-instance \
  --db-instance-identifier mydb \
  --engine mysql \
  --db-instance-class db.t2.micro
```

## ✅ Completion Checklist

- [ ] Created resources
- [ ] Configured services
- [ ] Tested functionality
- [ ] Documented setup

---

**Next**: Explore project ideas.

