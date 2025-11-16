# AWS CLI Cheat Sheet - Complete Reference

## 🔧 Configuration

### Setup

```bash
# Configure AWS CLI
aws configure
aws configure --profile production
aws configure list
aws configure list-profiles

# Set credentials
export AWS_ACCESS_KEY_ID=your-key
export AWS_SECRET_ACCESS_KEY=your-secret
export AWS_DEFAULT_REGION=us-east-1
export AWS_PROFILE=production

# View current identity
aws sts get-caller-identity
aws sts get-caller-identity --profile production

# Switch region
aws --region us-west-2 s3 ls
export AWS_DEFAULT_REGION=us-west-2
```

## 💻 EC2 (Elastic Compute Cloud)

### Instances

```bash
# List instances
aws ec2 describe-instances
aws ec2 describe-instances --filters "Name=instance-state-name,Values=running"
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,State.Name,PublicIpAddress]' --output table

# Launch instance
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t2.micro \
  --key-name my-key \
  --security-group-ids sg-12345678 \
  --subnet-id subnet-12345678

# Start/Stop/Terminate
aws ec2 start-instances --instance-ids i-1234567890abcdef0
aws ec2 stop-instances --instance-ids i-1234567890abcdef0
aws ec2 terminate-instances --instance-ids i-1234567890abcdef0

# Reboot
aws ec2 reboot-instances --instance-ids i-1234567890abcdef0

# Get instance details
aws ec2 describe-instances --instance-ids i-1234567890abcdef0
aws ec2 describe-instance-status --instance-ids i-1234567890abcdef0
```

### AMIs

```bash
# List AMIs
aws ec2 describe-images --owners self
aws ec2 describe-images --filters "Name=name,Values=my-ami-*"
aws ec2 describe-images --image-ids ami-12345678

# Create AMI
aws ec2 create-image \
  --instance-id i-1234567890abcdef0 \
  --name "my-ami-$(date +%Y%m%d)"

# Copy AMI
aws ec2 copy-image \
  --source-region us-east-1 \
  --source-image-id ami-12345678 \
  --name "copied-ami" \
  --region us-west-2
```

### Security Groups

```bash
# List security groups
aws ec2 describe-security-groups
aws ec2 describe-security-groups --group-ids sg-12345678

# Create security group
aws ec2 create-security-group \
  --group-name my-sg \
  --description "My security group" \
  --vpc-id vpc-12345678

# Add rule
aws ec2 authorize-security-group-ingress \
  --group-id sg-12345678 \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# Remove rule
aws ec2 revoke-security-group-ingress \
  --group-id sg-12345678 \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0
```

### Key Pairs

```bash
# List key pairs
aws ec2 describe-key-pairs

# Create key pair
aws ec2 create-key-pair --key-name my-key --query 'KeyMaterial' --output text > my-key.pem
chmod 400 my-key.pem

# Delete key pair
aws ec2 delete-key-pair --key-name my-key
```

## 📦 S3 (Simple Storage Service)

### Buckets

```bash
# List buckets
aws s3 ls
aws s3 ls s3://bucket-name

# Create bucket
aws s3 mb s3://bucket-name
aws s3 mb s3://bucket-name --region us-west-2

# Delete bucket
aws s3 rb s3://bucket-name
aws s3 rb s3://bucket-name --force  # With contents

# Bucket info
aws s3api get-bucket-location --bucket bucket-name
aws s3api get-bucket-versioning --bucket bucket-name
```

### Objects

```bash
# List objects
aws s3 ls s3://bucket-name
aws s3 ls s3://bucket-name/path/
aws s3 ls s3://bucket-name --recursive

# Copy file
aws s3 cp file.txt s3://bucket-name/
aws s3 cp file.txt s3://bucket-name/path/file.txt
aws s3 cp s3://bucket-name/file.txt ./local-file.txt

# Copy directory
aws s3 cp ./local-dir s3://bucket-name/dir/ --recursive
aws s3 sync ./local-dir s3://bucket-name/dir/

# Move file
aws s3 mv file.txt s3://bucket-name/
aws s3 mv s3://bucket-name/file1.txt s3://bucket-name/file2.txt

# Remove file
aws s3 rm s3://bucket-name/file.txt
aws s3 rm s3://bucket-name/dir/ --recursive

# Presigned URL
aws s3 presign s3://bucket-name/file.txt
aws s3 presign s3://bucket-name/file.txt --expires-in 3600
```

### S3 Operations

```bash
# Sync directories
aws s3 sync ./local s3://bucket-name/remote/
aws s3 sync s3://bucket-name/remote/ ./local/

# Website hosting
aws s3 website s3://bucket-name/ --index-document index.html
aws s3api put-bucket-website \
  --bucket bucket-name \
  --website-configuration file://website.json
```

## 🔐 IAM (Identity and Access Management)

### Users

```bash
# List users
aws iam list-users
aws iam get-user --user-name username

# Create user
aws iam create-user --user-name username

# Delete user
aws iam delete-user --user-name username

# List user policies
aws iam list-attached-user-policies --user-name username
aws iam list-user-policies --user-name username
```

### Groups

```bash
# List groups
aws iam list-groups

# Create group
aws iam create-group --group-name groupname

# Add user to group
aws iam add-user-to-group --user-name username --group-name groupname

# Attach policy to group
aws iam attach-group-policy \
  --group-name groupname \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

### Policies

```bash
# List policies
aws iam list-policies
aws iam list-policies --scope Local

# Get policy
aws iam get-policy --policy-arn arn:aws:iam::123456789012:policy/mypolicy

# Create policy
aws iam create-policy \
  --policy-name mypolicy \
  --policy-document file://policy.json

# Attach policy to user
aws iam attach-user-policy \
  --user-name username \
  --policy-arn arn:aws:iam::123456789012:policy/mypolicy
```

### Roles

```bash
# List roles
aws iam list-roles

# Create role
aws iam create-role \
  --role-name rolename \
  --assume-role-policy-document file://trust-policy.json

# Attach policy to role
aws iam attach-role-policy \
  --role-name rolename \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

## 🌐 VPC (Virtual Private Cloud)

### VPCs

```bash
# List VPCs
aws ec2 describe-vpcs
aws ec2 describe-vpcs --vpc-ids vpc-12345678

# Create VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# Delete VPC
aws ec2 delete-vpc --vpc-id vpc-12345678
```

### Subnets

```bash
# List subnets
aws ec2 describe-subnets
aws ec2 describe-subnets --filters "Name=vpc-id,Values=vpc-12345678"

# Create subnet
aws ec2 create-subnet \
  --vpc-id vpc-12345678 \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a

# Delete subnet
aws ec2 delete-subnet --subnet-id subnet-12345678
```

### Internet Gateways

```bash
# List internet gateways
aws ec2 describe-internet-gateways

# Create internet gateway
aws ec2 create-internet-gateway

# Attach to VPC
aws ec2 attach-internet-gateway \
  --internet-gateway-id igw-12345678 \
  --vpc-id vpc-12345678
```

### Route Tables

```bash
# List route tables
aws ec2 describe-route-tables

# Create route table
aws ec2 create-route-table --vpc-id vpc-12345678

# Add route
aws ec2 create-route \
  --route-table-id rtb-12345678 \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id igw-12345678
```

## 🗄️ RDS (Relational Database Service)

### Databases

```bash
# List databases
aws rds describe-db-instances

# Create database
aws rds create-db-instance \
  --db-instance-identifier mydb \
  --db-instance-class db.t2.micro \
  --engine mysql \
  --master-username admin \
  --master-user-password password \
  --allocated-storage 20

# Delete database
aws rds delete-db-instance \
  --db-instance-identifier mydb \
  --skip-final-snapshot
```

### Snapshots

```bash
# List snapshots
aws rds describe-db-snapshots

# Create snapshot
aws rds create-db-snapshot \
  --db-instance-identifier mydb \
  --db-snapshot-identifier mydb-snapshot
```

## 🔄 Lambda

### Functions

```bash
# List functions
aws lambda list-functions

# Get function
aws lambda get-function --function-name myfunction

# Invoke function
aws lambda invoke \
  --function-name myfunction \
  --payload '{"key":"value"}' \
  response.json

# Update function code
aws lambda update-function-code \
  --function-name myfunction \
  --zip-file fileb://function.zip
```

## ☸️ EKS (Elastic Kubernetes Service)

### Clusters

```bash
# List clusters
aws eks list-clusters

# Describe cluster
aws eks describe-cluster --name my-cluster

# Update kubeconfig
aws eks update-kubeconfig --name my-cluster --region us-east-1
```

## 📊 CloudWatch

### Logs

```bash
# List log groups
aws logs describe-log-groups

# Get log events
aws logs get-log-events \
  --log-group-name /aws/lambda/myfunction \
  --log-stream-name stream-name

# Tail logs
aws logs tail /aws/lambda/myfunction --follow
```

### Metrics

```bash
# List metrics
aws cloudwatch list-metrics

# Get metric statistics
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
  --start-time 2024-01-01T00:00:00Z \
  --end-time 2024-01-02T00:00:00Z \
  --period 3600 \
  --statistics Average
```

## 🔒 Secrets Manager

```bash
# List secrets
aws secretsmanager list-secrets

# Get secret
aws secretsmanager get-secret-value --secret-id mysecret

# Create secret
aws secretsmanager create-secret \
  --name mysecret \
  --secret-string '{"username":"admin","password":"secret123"}'
```

## 📝 Common Patterns

### Query and Filter

```bash
# Query with JMESPath
aws ec2 describe-instances \
  --query 'Reservations[*].Instances[*].[InstanceId,State.Name,PublicIpAddress]' \
  --output table

# Filter results
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  "Name=tag:Environment,Values=production"

# Output formats
aws s3 ls --output json
aws s3 ls --output table
aws s3 ls --output text
aws s3 ls --output yaml
```

### Pagination

```bash
# Use --max-items and --starting-token
aws s3api list-objects-v2 \
  --bucket bucket-name \
  --max-items 100 \
  --starting-token token
```

### Profiles

```bash
# Use profile
aws s3 ls --profile production
aws configure --profile production

# Switch profile
export AWS_PROFILE=production
```

## 🚨 Troubleshooting

```bash
# Check credentials
aws sts get-caller-identity

# Enable debug mode
export AWS_CLI_LOG_LEVEL=DEBUG
aws s3 ls

# Check region
aws configure get region

# Validate JSON
aws iam create-policy \
  --policy-name test \
  --policy-document file://policy.json \
  --dry-run
```

## 📚 Quick Reference

### Service Abbreviations

```bash
ec2    # Elastic Compute Cloud
s3     # Simple Storage Service
iam    # Identity and Access Management
rds    # Relational Database Service
lambda # AWS Lambda
eks    # Elastic Kubernetes Service
vpc    # Virtual Private Cloud
```

### Common Flags

```bash
--region          # Specify region
--profile         # Use profile
--output          # Output format (json, table, text, yaml)
--query           # JMESPath query
--filters         # Filter results
--dry-run         # Validate without executing
```

---

**Remember**: Always verify commands before executing destructive operations. Use `--dry-run` when available. Keep credentials secure and use IAM roles when possible.
