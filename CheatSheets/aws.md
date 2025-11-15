# AWS Cheat Sheet

## 📚 Essential Commands

### EC2
```bash
aws ec2 describe-instances  # List instances
aws ec2 run-instances --image-id ami-xxx  # Launch
aws ec2 terminate-instances --instance-ids i-xxx  # Terminate
```

### S3
```bash
aws s3 ls            # List buckets
aws s3 mb s3://bucket  # Create bucket
aws s3 cp file s3://bucket/  # Copy file
aws s3 sync dir s3://bucket/  # Sync directory
```

### IAM
```bash
aws iam list-users   # List users
aws iam create-user --user-name name  # Create user
aws iam attach-user-policy  # Attach policy
```

### General
```bash
aws configure        # Configure credentials
aws sts get-caller-identity  # Current identity
aws --region us-east-1  # Specify region
```

---

**Quick Reference**: Essential AWS CLI commands.

