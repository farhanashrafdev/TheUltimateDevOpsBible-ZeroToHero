# AWS Hands-On Labs: Complete Practical Guide

## 🎯 Introduction

This comprehensive guide provides hands-on AWS labs to master cloud infrastructure. Each lab includes step-by-step instructions and real-world scenarios.

## 📚 Lab Structure

Each lab includes:
- **Objective**: Learning goals
- **Prerequisites**: Required setup
- **Step-by-Step Tasks**: Detailed instructions
- **Terraform/CLI Examples**: Complete code
- **Verification**: Success criteria
- **Cleanup**: Resource cleanup
- **Challenges**: Advanced exercises

## ☁️ Lab 1: EC2 Fundamentals

### Objective
Launch and manage EC2 instances, understand security groups, and work with EBS volumes.

### Prerequisites
- AWS account
- AWS CLI configured
- Basic Linux knowledge

### Tasks

#### Task 1: Launch EC2 Instance

```bash
# Create key pair
aws ec2 create-key-pair \
  --key-name my-key \
  --query 'KeyMaterial' \
  --output text > my-key.pem
chmod 400 my-key.pem

# Get latest Amazon Linux 2 AMI
AMI_ID=$(aws ec2 describe-images \
  --owners amazon \
  --filters "Name=name,Values=amzn2-ami-hvm-*-x86_64-gp2" \
  --query 'Images | sort_by(@, &CreationDate) | [-1].ImageId' \
  --output text)

# Create security group
SG_ID=$(aws ec2 create-security-group \
  --group-name my-sg \
  --description "My security group" \
  --query 'GroupId' \
  --output text)

# Add SSH rule
aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0

# Launch instance
INSTANCE_ID=$(aws ec2 run-instances \
  --image-id $AMI_ID \
  --instance-type t2.micro \
  --key-name my-key \
  --security-group-ids $SG_ID \
  --query 'Instances[0].InstanceId' \
  --output text)

# Wait for running
aws ec2 wait instance-running --instance-ids $INSTANCE_ID

# Get public IP
PUBLIC_IP=$(aws ec2 describe-instances \
  --instance-ids $INSTANCE_ID \
  --query 'Reservations[0].Instances[0].PublicIpAddress' \
  --output text)

echo "Instance IP: $PUBLIC_IP"
```

#### Task 2: Connect and Configure

```bash
# Connect via SSH
ssh -i my-key.pem ec2-user@$PUBLIC_IP

# Inside instance
sudo yum update -y
sudo yum install -y nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```

#### Task 3: Create and Attach EBS Volume

```bash
# Create EBS volume
VOLUME_ID=$(aws ec2 create-volume \
  --availability-zone us-east-1a \
  --size 10 \
  --volume-type gp3 \
  --query 'VolumeId' \
  --output text)

# Wait for available
aws ec2 wait volume-available --volume-ids $VOLUME_ID

# Attach volume
aws ec2 attach-volume \
  --volume-id $VOLUME_ID \
  --instance-id $INSTANCE_ID \
  --device /dev/sdf

# Inside instance (after SSH)
sudo mkfs -t xfs /dev/xvdf
sudo mkdir /data
sudo mount /dev/xvdf /data
echo '/dev/xvdf /data xfs defaults,nofail 0 2' | sudo tee -a /etc/fstab
```

#### Task 4: Create AMI

```bash
# Create AMI
AMI_ID=$(aws ec2 create-image \
  --instance-id $INSTANCE_ID \
  --name "my-custom-ami-$(date +%Y%m%d)" \
  --description "Custom AMI with nginx" \
  --query 'ImageId' \
  --output text)

# Wait for available
aws ec2 wait image-available --image-ids $AMI_ID
```

### Verification

```bash
# Check instance
aws ec2 describe-instances --instance-ids $INSTANCE_ID

# Test web server
curl http://$PUBLIC_IP

# Check volume
aws ec2 describe-volumes --volume-ids $VOLUME_ID
```

### Cleanup

```bash
# Terminate instance
aws ec2 terminate-instances --instance-ids $INSTANCE_ID
aws ec2 wait instance-terminated --instance-ids $INSTANCE_ID

# Delete volume
aws ec2 delete-volume --volume-id $VOLUME_ID

# Delete security group
aws ec2 delete-security-group --group-id $SG_ID

# Delete key pair
aws ec2 delete-key-pair --key-name my-key
```

## 📦 Lab 2: S3 Operations

### Objective
Work with S3 buckets, objects, versioning, and lifecycle policies.

### Tasks

#### Task 1: Create and Configure Bucket

```bash
# Create bucket
BUCKET_NAME="my-bucket-$(date +%s)"
aws s3 mb s3://$BUCKET_NAME

# Enable versioning
aws s3api put-bucket-versioning \
  --bucket $BUCKET_NAME \
  --versioning-configuration Status=Enabled

# Enable encryption
aws s3api put-bucket-encryption \
  --bucket $BUCKET_NAME \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "AES256"
      }
    }]
  }'

# Block public access
aws s3api put-public-access-block \
  --bucket $BUCKET_NAME \
  --public-access-block-configuration \
    "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"
```

#### Task 2: Upload and Manage Objects

```bash
# Create test file
echo "Hello World" > test.txt

# Upload file
aws s3 cp test.txt s3://$BUCKET_NAME/

# Upload directory
aws s3 cp ./data/ s3://$BUCKET_NAME/data/ --recursive

# List objects
aws s3 ls s3://$BUCKET_NAME/

# Download file
aws s3 cp s3://$BUCKET_NAME/test.txt downloaded.txt

# Sync directory
aws s3 sync ./local/ s3://$BUCKET_NAME/remote/
```

#### Task 3: Lifecycle Policies

```bash
# Create lifecycle policy
cat > lifecycle.json <<EOF
{
  "Rules": [
    {
      "Id": "Transition to IA",
      "Status": "Enabled",
      "Prefix": "",
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "STANDARD_IA"
        },
        {
          "Days": 90,
          "StorageClass": "GLACIER"
        }
      ],
      "Expiration": {
        "Days": 365
      }
    }
  ]
}
EOF

aws s3api put-bucket-lifecycle-configuration \
  --bucket $BUCKET_NAME \
  --lifecycle-configuration file://lifecycle.json
```

#### Task 4: Static Website Hosting

```bash
# Create index.html
cat > index.html <<EOF
<html>
  <body>
    <h1>Hello from S3!</h1>
  </body>
</html>
EOF

# Upload
aws s3 cp index.html s3://$BUCKET_NAME/

# Enable static website hosting
aws s3 website s3://$BUCKET_NAME/ \
  --index-document index.html

# Get website URL
aws s3api get-bucket-website \
  --bucket $BUCKET_NAME
```

### Verification

```bash
# List bucket
aws s3 ls s3://$BUCKET_NAME/

# Check versioning
aws s3api get-bucket-versioning --bucket $BUCKET_NAME

# Check encryption
aws s3api get-bucket-encryption --bucket $BUCKET_NAME
```

### Cleanup

```bash
# Delete all objects
aws s3 rm s3://$BUCKET_NAME/ --recursive

# Delete bucket
aws s3 rb s3://$BUCKET_NAME
```

## 🌐 Lab 3: VPC Setup

### Objective
Create a complete VPC with public and private subnets, NAT gateway, and security groups.

### Tasks

#### Task 1: Create VPC with Terraform

```hcl
# vpc.tf
provider "aws" {
  region = "us-east-1"
}

# VPC
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = {
    Name = "main-vpc"
  }
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = {
    Name = "main-igw"
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
    Name = "public-subnet-${count.index + 1}"
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
    Name = "private-subnet-${count.index + 1}"
    Type = "Private"
  }
}

# NAT Gateway
resource "aws_eip" "nat" {
  domain = "vpc"
  
  tags = {
    Name = "nat-eip"
  }
}

resource "aws_nat_gateway" "main" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public[0].id
  
  tags = {
    Name = "main-nat"
  }
  
  depends_on = [aws_internet_gateway.main]
}

# Route Tables
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

resource "aws_route_table" "private" {
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.main.id
  }
  
  tags = {
    Name = "private-rt"
  }
}

# Route Table Associations
resource "aws_route_table_association" "public" {
  count          = 2
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "private" {
  count          = 2
  subnet_id      = aws_subnet.private[count.index].id
  route_table_id = aws_route_table.private.id
}

# Data source
data "aws_availability_zones" "available" {
  state = "available"
}
```

```bash
# Initialize Terraform
terraform init

# Plan
terraform plan

# Apply
terraform apply
```

#### Task 2: Launch Instances

```bash
# Launch in public subnet
aws ec2 run-instances \
  --image-id $AMI_ID \
  --instance-type t2.micro \
  --subnet-id <public-subnet-id> \
  --security-group-ids $SG_ID \
  --key-name my-key

# Launch in private subnet
aws ec2 run-instances \
  --image-id $AMI_ID \
  --instance-type t2.micro \
  --subnet-id <private-subnet-id> \
  --security-group-ids $SG_ID \
  --key-name my-key
```

### Verification

```bash
# Check VPC
aws ec2 describe-vpcs --filters "Name=tag:Name,Values=main-vpc"

# Check subnets
aws ec2 describe-subnets --filters "Name=vpc-id,Values=<vpc-id>"

# Test connectivity
# Public instance: Should have internet access
# Private instance: Should have outbound internet via NAT
```

## 🗄️ Lab 4: RDS Database

### Objective
Create and manage RDS database instances.

### Tasks

#### Task 1: Create RDS Instance

```bash
# Create DB subnet group
aws rds create-db-subnet-group \
  --db-subnet-group-name my-db-subnet-group \
  --db-subnet-group-description "My DB subnet group" \
  --subnet-ids <subnet1-id> <subnet2-id>

# Create security group for RDS
RDS_SG_ID=$(aws ec2 create-security-group \
  --group-name rds-sg \
  --description "RDS security group" \
  --vpc-id <vpc-id> \
  --query 'GroupId' \
  --output text)

# Allow MySQL from application security group
aws ec2 authorize-security-group-ingress \
  --group-id $RDS_SG_ID \
  --protocol tcp \
  --port 3306 \
  --source-group $APP_SG_ID

# Create RDS instance
aws rds create-db-instance \
  --db-instance-identifier mydb \
  --db-instance-class db.t3.micro \
  --engine mysql \
  --engine-version 8.0 \
  --master-username admin \
  --master-user-password MyPassword123! \
  --allocated-storage 20 \
  --storage-type gp3 \
  --vpc-security-group-ids $RDS_SG_ID \
  --db-subnet-group-name my-db-subnet-group \
  --backup-retention-period 7 \
  --multi-az \
  --storage-encrypted

# Wait for available
aws rds wait db-instance-available --db-instance-identifier mydb
```

#### Task 2: Create Read Replica

```bash
# Create read replica
aws rds create-db-instance-read-replica \
  --db-instance-identifier mydb-replica \
  --source-db-instance-identifier mydb \
  --db-instance-class db.t3.micro \
  --publicly-accessible
```

#### Task 3: Create Snapshot

```bash
# Create manual snapshot
aws rds create-db-snapshot \
  --db-instance-identifier mydb \
  --db-snapshot-identifier mydb-snapshot-$(date +%Y%m%d)

# Wait for snapshot
aws rds wait db-snapshot-completed \
  --db-snapshot-identifier mydb-snapshot-$(date +%Y%m%d)
```

### Verification

```bash
# Check RDS instance
aws rds describe-db-instances --db-instance-identifier mydb

# Get endpoint
ENDPOINT=$(aws rds describe-db-instances \
  --db-instance-identifier mydb \
  --query 'DBInstances[0].Endpoint.Address' \
  --output text)

# Test connection (from EC2 instance)
mysql -h $ENDPOINT -u admin -p
```

### Cleanup

```bash
# Delete read replica
aws rds delete-db-instance \
  --db-instance-identifier mydb-replica \
  --skip-final-snapshot

# Delete main instance
aws rds delete-db-instance \
  --db-instance-identifier mydb \
  --skip-final-snapshot
```

## ✅ Completion Checklist

- [ ] Launched EC2 instances
- [ ] Created S3 buckets
- [ ] Set up VPC
- [ ] Created RDS database
- [ ] Tested connectivity
- [ ] Cleaned up resources
- [ ] Completed challenges

---

**Next**: Complete Terraform labs and advanced AWS scenarios.

**Remember**: AWS labs provide hands-on experience. Always clean up resources to avoid charges. Practice with free tier services first, then explore advanced features. Good AWS skills enable cloud-native infrastructure.
