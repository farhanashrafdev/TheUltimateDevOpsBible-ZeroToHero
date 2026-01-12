# AWS Interview Questions

## 🎯 Core Services

**Q: Explain VPC and its components.**

**A:**
- **VPC**: Isolated virtual network
- **Subnets**: IP ranges (public/private)
- **Route Tables**: Traffic routing rules
- **Internet Gateway**: Public internet access
- **NAT Gateway**: Private subnet outbound access
- **Security Groups**: Instance-level firewall (stateful)
- **NACLs**: Subnet-level firewall (stateless)

**Q: Difference between Security Groups and NACLs?**

| Feature | Security Group | NACL |
|---------|---------------|------|
| Level | Instance | Subnet |
| State | Stateful | Stateless |
| Rules | Allow only | Allow + Deny |
| Default | Deny all inbound | Allow all |

**Q: Explain EC2 instance types.**

**A:**
- **General Purpose (T, M)**: Balanced compute/memory
- **Compute Optimized (C)**: High CPU
- **Memory Optimized (R, X)**: Large memory workloads
- **Storage Optimized (I, D)**: High disk I/O

**Q: What's the difference between EBS and Instance Store?**

| Feature | EBS | Instance Store |
|---------|-----|----------------|
| Persistence | Survives stop/terminate | Lost on stop |
| Backup | Snapshots to S3 | Manual |
| Size | Up to 16TB | Fixed |
| Performance | Provisioned IOPS | Very high |

## 📦 Storage & Databases

**Q: S3 storage classes and use cases?**

**A:**
- **Standard**: Frequently accessed
- **Standard-IA**: Infrequent access, quick retrieval
- **One Zone-IA**: Non-critical infrequent data
- **Glacier**: Archive (minutes to hours retrieval)
- **Glacier Deep Archive**: Long-term archive (12+ hours)

**Q: RDS vs DynamoDB?**

| Feature | RDS | DynamoDB |
|---------|-----|----------|
| Type | Relational | NoSQL |
| Scaling | Vertical | Horizontal |
| Schema | Fixed | Flexible |
| Use Case | Complex queries | Key-value/document |

**Q: Explain RDS Multi-AZ vs Read Replicas.**

**A:**
- **Multi-AZ**: High availability, automatic failover, same region
- **Read Replicas**: Read scaling, can be cross-region, async replication

## 🔐 Security & IAM

**Q: Explain IAM roles vs users.**

**A:**
- **Users**: Long-term credentials for people
- **Roles**: Temporary credentials for services/applications

**Q: What is an IAM policy structure?**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": "arn:aws:s3:::bucket/*",
      "Condition": {
        "IpAddress": {"aws:SourceIp": "10.0.0.0/8"}
      }
    }
  ]
}
```

**Q: How do you secure S3 buckets?**

**A:**
- Block public access (account/bucket level)
- Bucket policies with least privilege
- Enable encryption (SSE-S3, SSE-KMS)
- Enable versioning and MFA delete
- Enable access logging
- Use VPC endpoints for private access

## 🌐 Networking & Load Balancing

**Q: Explain ALB vs NLB vs CLB.**

| Feature | ALB | NLB | CLB |
|---------|-----|-----|-----|
| Layer | 7 (HTTP) | 4 (TCP/UDP) | 4+7 |
| Routing | Path/host | Connection | Basic |
| Performance | Good | Ultra-low latency | Legacy |

**Q: How does Route 53 routing work?**

**A:**
- **Simple**: Single resource
- **Weighted**: Percentage distribution
- **Latency**: Lowest latency region
- **Failover**: Primary/secondary
- **Geolocation**: By user location

## 🎯 Scenario Questions

**Q: Design a highly available web application.**

**A:**
```
┌─────────────────────────────────────────┐
│ Route 53 (DNS)                          │
└────────────────┬────────────────────────┘
                 │
         ┌───────┴───────┐
         ▼               ▼
    ┌─────────┐    ┌─────────┐
    │  ALB    │    │  ALB    │
    │ (AZ-a)  │    │ (AZ-b)  │
    └────┬────┘    └────┬────┘
         │               │
    ┌────┴────┐    ┌────┴────┐
    │ EC2 ASG │    │ EC2 ASG │
    └────┬────┘    └────┬────┘
         │               │
         └───────┬───────┘
                 ▼
           ┌───────────┐
           │ RDS       │
           │ Multi-AZ  │
           └───────────┘
```

**Q: How would you migrate an on-premises database to AWS?**

**A:**
1. Assess: AWS Database Migration Service (DMS)
2. Schema conversion: AWS SCT
3. Migrate: DMS with CDC for minimal downtime
4. Validate: Compare source and target
5. Cutover: Switch applications

**Q: An EC2 instance can't reach the internet. Debug steps?**

**A:**
1. Check VPC/subnet configuration
2. Verify route table has IGW route (0.0.0.0/0)
3. Check security group outbound rules
4. Check NACL rules
5. Verify Elastic IP/public IP assigned
6. Check instance status checks

---

**Next**: Review [DevSecOps Interview](./devsecops.md) questions.

