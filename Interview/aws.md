# AWS Interview Questions: Complete Guide

## 🎯 Introduction

This comprehensive guide covers AWS interview questions from basic services to advanced architecture design.

## 💻 Compute Services

### Explain EC2

**Answer**: Elastic Compute Cloud provides resizable virtual servers:

**Key Features**:
- Virtual servers (instances)
- Multiple instance types
- AMIs (Amazon Machine Images)
- Security groups (firewall)
- EBS volumes (storage)
- Auto Scaling
- Elastic IPs

**Instance Types**:
- **General Purpose**: t3, m5 (balanced)
- **Compute Optimized**: c5 (high CPU)
- **Memory Optimized**: r5 (high RAM)
- **Storage Optimized**: i3 (high IOPS)
- **GPU**: p3, g4 (GPU computing)

**Example Use Case**: Web servers, application servers, databases

### What is Lambda?

**Answer**: AWS Lambda is serverless compute:

**Features**:
- Pay per request
- Automatic scaling
- No server management
- Event-driven
- Supports multiple languages

**Use Cases**:
- API backends
- Data processing
- Real-time file processing
- Scheduled tasks
- Event-driven applications

**Limitations**:
- 15-minute timeout
- Memory: 128 MB to 10 GB
- Ephemeral storage: 512 MB to 10 GB

### Explain ECS and EKS

**Answer**:

**ECS (Elastic Container Service)**:
- Fully managed container orchestration
- Supports Docker containers
- Launch types: Fargate (serverless) or EC2
- Integrated with AWS services

**EKS (Elastic Kubernetes Service)**:
- Managed Kubernetes service
- Compatible with Kubernetes APIs
- Runs on EC2 or Fargate
- Integrates with AWS services

**When to use**:
- ECS: AWS-native, simpler
- EKS: Kubernetes standard, multi-cloud

## 📦 Storage Services

### What is S3?

**Answer**: Simple Storage Service for object storage:

**Key Features**:
- Unlimited storage
- 99.999999999% (11 9's) durability
- Versioning
- Lifecycle policies
- Encryption (SSE-S3, SSE-KMS, SSE-C)
- Access control (IAM, bucket policies, ACLs)
- Static website hosting

**Storage Classes**:
- **Standard**: General purpose
- **Standard-IA**: Infrequently accessed
- **Glacier**: Archive storage
- **Intelligent-Tiering**: Automatic optimization

**Use Cases**:
- Backup and archive
- Static website hosting
- Data lakes
- Content distribution

### Explain EBS

**Answer**: Elastic Block Store provides block storage for EC2:

**Volume Types**:
- **gp3**: General Purpose SSD (latest, best price/performance)
- **gp2**: General Purpose SSD (legacy)
- **io1/io2**: Provisioned IOPS SSD
- **st1**: Throughput Optimized HDD
- **sc1**: Cold HDD

**Features**:
- Persistent storage
- Snapshots (backup)
- Encryption
- Multi-attach (io1/io2)

### What is EFS?

**Answer**: Elastic File System provides managed NFS:

**Features**:
- Shared file system
- Multiple EC2 instances can access
- Automatic scaling
- Encryption
- Lifecycle management

**Use Cases**:
- Shared application storage
- Content management
- Web serving
- Database backups

## 🌐 Networking

### Explain VPC

**Answer**: Virtual Private Cloud provides isolated network:

**Components**:
- **Subnets**: Network segments (public/private)
- **Route Tables**: Routing rules
- **Internet Gateway**: Internet access
- **NAT Gateway**: Outbound internet for private subnets
- **Security Groups**: Instance-level firewall
- **Network ACLs**: Subnet-level firewall
- **VPC Peering**: Connect VPCs
- **VPN/Direct Connect**: On-premises connectivity

**Best Practices**:
- Use private subnets for resources
- Public subnets for load balancers
- NAT Gateway for outbound internet
- Security groups for access control

### What is CloudFront?

**Answer**: Content Delivery Network (CDN):

**Features**:
- Global edge locations
- Caching
- DDoS protection
- SSL/TLS termination
- Origin failover
- Signed URLs

**Use Cases**:
- Static website hosting
- Video streaming
- API acceleration
- Global content delivery

### Explain Route 53

**Answer**: Scalable DNS service:

**Features**:
- Domain registration
- DNS hosting
- Health checks
- Routing policies (simple, weighted, latency, failover, geolocation)
- Private hosted zones

**Routing Policies**:
- **Simple**: Standard DNS
- **Weighted**: Distribute traffic
- **Latency**: Route to lowest latency
- **Failover**: Active-passive
- **Geolocation**: Route by location

## 🗄️ Databases

### Explain RDS

**Answer**: Relational Database Service provides managed databases:

**Supported Engines**:
- MySQL, PostgreSQL, MariaDB
- Oracle, SQL Server
- Aurora (MySQL/PostgreSQL compatible)

**Features**:
- Multi-AZ (high availability)
- Read replicas (scalability)
- Automated backups
- Encryption at rest and in transit
- Automated patching
- Monitoring (CloudWatch)

**Use Cases**:
- Web applications
- Enterprise applications
- OLTP workloads

### What is DynamoDB?

**Answer**: Fully managed NoSQL database:

**Features**:
- Serverless
- Auto-scaling
- Single-digit millisecond latency
- Global tables (multi-region)
- Streams (change data capture)
- On-demand or provisioned capacity

**Use Cases**:
- Gaming applications
- IoT applications
- Mobile applications
- Real-time applications

### Explain ElastiCache

**Answer**: In-memory caching service:

**Engines**:
- **Redis**: Advanced data structures
- **Memcached**: Simple key-value

**Use Cases**:
- Session storage
- Database caching
- Real-time analytics
- Leaderboards

## 🔒 Security

### What is IAM?

**Answer**: Identity and Access Management:

**Components**:
- **Users**: Individual accounts
- **Groups**: Collections of users
- **Roles**: Temporary credentials
- **Policies**: Permissions (JSON)

**Best Practices**:
- Least privilege
- Use roles, not users for applications
- Enable MFA
- Regular access reviews
- Use policy conditions

### Explain Security Groups vs NACLs

**Answer**:

**Security Groups**:
- Instance-level firewall
- Stateful (return traffic allowed)
- Allow rules only
- Evaluates all rules

**Network ACLs**:
- Subnet-level firewall
- Stateless (explicit allow/deny)
- Allow and deny rules
- Evaluated in order

### What is KMS?

**Answer**: Key Management Service:

**Features**:
- Encryption key management
- Key rotation
- Integration with AWS services
- CloudHSM for FIPS 140-2 Level 3

**Use Cases**:
- Encrypt S3 objects
- Encrypt EBS volumes
- Encrypt RDS databases
- Application-level encryption

## 📊 Monitoring

### Explain CloudWatch

**Answer**: Monitoring and observability service:

**Features**:
- Metrics (CPU, memory, custom)
- Logs (centralized logging)
- Alarms (notifications)
- Dashboards (visualization)
- Events (event-driven automation)

**Key Metrics**:
- EC2: CPU, Network, Disk
- RDS: CPU, Connections, Storage
- Lambda: Invocations, Duration, Errors

### What is CloudTrail?

**Answer**: API logging and auditing:

**Features**:
- Logs all API calls
- Who, what, when, where
- Compliance and auditing
- Security analysis
- Resource change tracking

## 🎯 Scenario Questions

### Design a highly available application on AWS

**Answer**:
1. **Multi-AZ Deployment**:
   - Deploy across multiple availability zones
   - Use Application Load Balancer

2. **Auto Scaling**:
   - Auto Scaling Groups
   - Scale based on metrics

3. **Database**:
   - RDS Multi-AZ
   - Read replicas

4. **CDN**:
   - CloudFront for static content

5. **DNS**:
   - Route 53 with health checks

6. **Monitoring**:
   - CloudWatch alarms
   - Automated responses

### How do you secure AWS resources?

**Answer**:
1. **IAM**:
   - Least privilege
   - Use roles
   - Enable MFA

2. **Network**:
   - VPC with private subnets
   - Security groups
   - Network ACLs

3. **Encryption**:
   - At rest: KMS
   - In transit: TLS/SSL

4. **Monitoring**:
   - CloudTrail (audit)
   - CloudWatch (monitoring)
   - AWS Config (compliance)

5. **Additional**:
   - WAF (Web Application Firewall)
   - Shield (DDoS protection)
   - GuardDuty (threat detection)

### Explain cost optimization strategies

**Answer**:
1. **Right Sizing**:
   - Match instance to workload
   - Use CloudWatch metrics

2. **Reserved Instances**:
   - 1-3 year commitments
   - Up to 72% savings

3. **Spot Instances**:
   - Up to 90% savings
   - For flexible workloads

4. **Storage**:
   - S3 lifecycle policies
   - Glacier for archives
   - Delete unused resources

5. **Auto Scaling**:
   - Scale down when not needed
   - Use scheduled scaling

## ✅ Key Areas to Master

- [ ] EC2 and compute services
- [ ] S3 and storage services
- [ ] VPC and networking
- [ ] RDS and databases
- [ ] IAM and security
- [ ] CloudWatch and monitoring
- [ ] Cost optimization
- [ ] High availability design
- [ ] Security best practices
- [ ] Architecture patterns

---

**Remember**: AWS interviews test both service knowledge and architecture design. Be prepared to explain services, design systems, and discuss trade-offs. Practice with AWS free tier and build projects.
