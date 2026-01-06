# Multi-Cloud Comparison: AWS vs GCP vs Azure

## 🎯 Overview

This guide provides a comprehensive comparison of the three major cloud providers, helping you understand service mappings and make informed decisions.

## 🗺️ Service Mapping

### Compute Services

```yaml
┌────────────────────┬─────────────────────┬─────────────────────┬─────────────────────┐
│ Category           │ AWS                 │ GCP                 │ Azure               │
├────────────────────┼─────────────────────┼─────────────────────┼─────────────────────┤
│ Virtual Machines   │ EC2                 │ Compute Engine      │ Virtual Machines    │
│ Containers (K8s)   │ EKS                 │ GKE                 │ AKS                 │
│ Serverless Compute │ Lambda              │ Cloud Functions     │ Azure Functions     │
│ Container Service  │ ECS/Fargate         │ Cloud Run           │ Container Instances │
│ Batch Processing   │ AWS Batch           │ Cloud Batch         │ Azure Batch         │
│ Spot/Preemptible   │ Spot Instances      │ Preemptible VMs     │ Spot VMs            │
└────────────────────┴─────────────────────┴─────────────────────┴─────────────────────┘
```

### Storage Services

```yaml
┌────────────────────┬─────────────────────┬─────────────────────┬─────────────────────┐
│ Category           │ AWS                 │ GCP                 │ Azure               │
├────────────────────┼─────────────────────┼─────────────────────┼─────────────────────┤
│ Object Storage     │ S3                  │ Cloud Storage       │ Blob Storage        │
│ Block Storage      │ EBS                 │ Persistent Disk     │ Managed Disks       │
│ File Storage       │ EFS                 │ Filestore           │ Azure Files         │
│ Archive Storage    │ S3 Glacier          │ Archive Storage     │ Archive Storage     │
│ Hybrid Storage     │ Storage Gateway     │ Transfer Appliance  │ StorSimple          │
└────────────────────┴─────────────────────┴─────────────────────┴─────────────────────┘
```

### Database Services

```yaml
┌────────────────────┬─────────────────────┬─────────────────────┬─────────────────────┐
│ Category           │ AWS                 │ GCP                 │ Azure               │
├────────────────────┼─────────────────────┼─────────────────────┼─────────────────────┤
│ Managed SQL        │ RDS                 │ Cloud SQL           │ Azure SQL           │
│ PostgreSQL         │ RDS/Aurora          │ Cloud SQL/AlloyDB   │ Azure PostgreSQL    │
│ MySQL              │ RDS/Aurora          │ Cloud SQL           │ Azure MySQL         │
│ NoSQL Document     │ DynamoDB            │ Firestore           │ Cosmos DB           │
│ In-Memory Cache    │ ElastiCache         │ Memorystore         │ Cache for Redis     │
│ Data Warehouse     │ Redshift            │ BigQuery            │ Synapse Analytics   │
│ Time Series        │ Timestream          │ Cloud Bigtable      │ Time Series Insights│
└────────────────────┴─────────────────────┴─────────────────────┴─────────────────────┘
```

### Networking Services

```yaml
┌────────────────────┬─────────────────────┬─────────────────────┬─────────────────────┐
│ Category           │ AWS                 │ GCP                 │ Azure               │
├────────────────────┼─────────────────────┼─────────────────────┼─────────────────────┤
│ Virtual Network    │ VPC                 │ VPC                 │ Virtual Network     │
│ Load Balancer      │ ALB/NLB/CLB         │ Cloud Load Balancer │ Load Balancer       │
│ CDN                │ CloudFront          │ Cloud CDN           │ CDN                 │
│ DNS                │ Route 53            │ Cloud DNS           │ Azure DNS           │
│ VPN                │ VPN Gateway         │ Cloud VPN           │ VPN Gateway         │
│ Direct Connect     │ Direct Connect      │ Dedicated Interconnect│ExpressRoute       │
│ API Gateway        │ API Gateway         │ API Gateway         │ API Management      │
│ Service Mesh       │ App Mesh            │ Anthos Service Mesh │ Service Fabric Mesh │
└────────────────────┴─────────────────────┴─────────────────────┴─────────────────────┘
```

### Security & Identity

```yaml
┌────────────────────┬─────────────────────┬─────────────────────┬─────────────────────┐
│ Category           │ AWS                 │ GCP                 │ Azure               │
├────────────────────┼─────────────────────┼─────────────────────┼─────────────────────┤
│ Identity & Access  │ IAM                 │ IAM                 │ Azure AD/Entra ID   │
│ Secrets Management │ Secrets Manager     │ Secret Manager      │ Key Vault           │
│ Key Management     │ KMS                 │ Cloud KMS           │ Key Vault           │
│ Web Application FW │ WAF                 │ Cloud Armor         │ WAF                 │
│ DDoS Protection    │ Shield              │ Cloud Armor         │ DDoS Protection     │
│ Certificate Mgmt   │ ACM                 │ Certificate Manager │ App Service Certs   │
│ Security Center    │ Security Hub        │ Security Command Ctr│ Defender for Cloud  │
└────────────────────┴─────────────────────┴─────────────────────┴─────────────────────┘
```

### DevOps & CI/CD

```yaml
┌────────────────────┬─────────────────────┬─────────────────────┬─────────────────────┐
│ Category           │ AWS                 │ GCP                 │ Azure               │
├────────────────────┼─────────────────────┼─────────────────────┼─────────────────────┤
│ Source Control     │ CodeCommit          │ Cloud Source Repos  │ Azure Repos         │
│ CI/CD Pipeline     │ CodePipeline        │ Cloud Build         │ Azure Pipelines     │
│ Build Service      │ CodeBuild           │ Cloud Build         │ Azure Pipelines     │
│ Artifact Registry  │ ECR/CodeArtifact    │ Artifact Registry   │ Container Registry  │
│ IaC                │ CloudFormation      │ Deployment Manager  │ ARM/Bicep           │
│ Monitoring         │ CloudWatch          │ Cloud Monitoring    │ Azure Monitor       │
│ Logging            │ CloudWatch Logs     │ Cloud Logging       │ Log Analytics       │
│ Tracing            │ X-Ray               │ Cloud Trace         │ Application Insights│
└────────────────────┴─────────────────────┴─────────────────────┴─────────────────────┘
```

## 💻 Kubernetes Comparison

### Managed Kubernetes Services

```yaml
EKS (AWS):
  Pros:
    - Deep AWS integration
    - Mature ecosystem
    - Large community
    - Many add-ons available
  
  Cons:
    - Complex IAM integration
    - Control plane costs ($72/month)
    - Add-ons managed separately
  
  Best For:
    - AWS-heavy organizations
    - Complex networking requirements
    - Enterprise compliance needs

GKE (Google):
  Pros:
    - Most mature managed K8s
    - Autopilot mode (serverless nodes)
    - Free control plane (standard mode)
    - Advanced features included
  
  Cons:
    - GCP-specific networking concepts
    - Less flexibility in some areas
  
  Best For:
    - Kubernetes-first organizations
    - Simplicity preference
    - AI/ML workloads

AKS (Azure):
  Pros:
    - Free control plane
    - Good Azure AD integration
    - Azure Arc for hybrid
    - Windows container support
  
  Cons:
    - Less mature than GKE
    - Slower feature adoption
  
  Best For:
    - Microsoft shops
    - Windows workloads
    - Hybrid cloud with Azure Arc
```

### Kubernetes Setup Examples

#### AWS EKS

```bash
# Create EKS cluster with eksctl
eksctl create cluster \
  --name production \
  --region us-east-1 \
  --version 1.28 \
  --nodegroup-name workers \
  --node-type m5.large \
  --nodes 3 \
  --nodes-min 2 \
  --nodes-max 10 \
  --managed

# Configure kubectl
aws eks update-kubeconfig --name production --region us-east-1

# Install AWS Load Balancer Controller
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=production
```

#### GCP GKE

```bash
# Create GKE Autopilot cluster
gcloud container clusters create-auto production \
  --region us-central1 \
  --release-channel regular

# Or standard cluster with more control
gcloud container clusters create production \
  --region us-central1 \
  --num-nodes 3 \
  --machine-type e2-standard-4 \
  --enable-autoscaling \
  --min-nodes 2 \
  --max-nodes 10

# Get credentials
gcloud container clusters get-credentials production --region us-central1
```

#### Azure AKS

```bash
# Create AKS cluster
az aks create \
  --resource-group production-rg \
  --name production \
  --node-count 3 \
  --node-vm-size Standard_D4s_v3 \
  --enable-cluster-autoscaler \
  --min-count 2 \
  --max-count 10 \
  --kubernetes-version 1.28.0

# Get credentials
az aks get-credentials --resource-group production-rg --name production
```

## 💰 Cost Comparison

### Compute Pricing (On-Demand, US East)

```yaml
4 vCPU, 16GB RAM (Monthly):
  AWS (m5.xlarge):     ~$140
  GCP (e2-standard-4): ~$97
  Azure (D4s_v3):      ~$140

8 vCPU, 32GB RAM (Monthly):
  AWS (m5.2xlarge):    ~$280
  GCP (e2-standard-8): ~$194
  Azure (D8s_v3):      ~$280

Note: GCP typically 20-40% cheaper for similar specs
      Use reserved/committed use for 40-60% savings
```

### Object Storage Pricing (per GB/month)

```yaml
Standard Storage:
  AWS S3:              $0.023
  GCP Cloud Storage:   $0.020
  Azure Blob:          $0.0184

Infrequent Access:
  AWS S3-IA:           $0.0125
  GCP Nearline:        $0.010
  Azure Cool:          $0.01

Archive:
  AWS Glacier:         $0.004
  GCP Archive:         $0.0012
  Azure Archive:       $0.002
```

### Data Transfer Pricing

```yaml
Egress (Internet, per GB):
  First 10TB:
    AWS:   $0.09
    GCP:   $0.12
    Azure: $0.087

  10-50TB:
    AWS:   $0.085
    GCP:   $0.11
    Azure: $0.083

Note: Ingress is free on all providers
      Cross-region transfer varies significantly
```

## 🔧 IaC Examples

### Terraform Multi-Cloud

```hcl
# providers.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

# AWS Provider
provider "aws" {
  region = "us-east-1"
}

# GCP Provider
provider "google" {
  project = "my-project"
  region  = "us-central1"
}

# Azure Provider
provider "azurerm" {
  features {}
}
```

```hcl
# Object Storage - All Three Clouds
# AWS S3
resource "aws_s3_bucket" "data" {
  bucket = "my-data-bucket"
  
  tags = {
    Environment = "production"
  }
}

resource "aws_s3_bucket_versioning" "data" {
  bucket = aws_s3_bucket.data.id
  versioning_configuration {
    status = "Enabled"
  }
}

# GCP Cloud Storage
resource "google_storage_bucket" "data" {
  name     = "my-data-bucket"
  location = "US"
  
  versioning {
    enabled = true
  }
  
  labels = {
    environment = "production"
  }
}

# Azure Blob Storage
resource "azurerm_storage_account" "data" {
  name                     = "mydatastorageacct"
  resource_group_name      = azurerm_resource_group.main.name
  location                 = azurerm_resource_group.main.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
  
  blob_properties {
    versioning_enabled = true
  }
  
  tags = {
    environment = "production"
  }
}
```

## 📊 When to Choose Which

### Choose AWS When

```yaml
Ideal For:
  - Largest service catalog needed
  - Mature enterprise features required
  - Strong partner ecosystem important
  - Deep integration with existing AWS services
  - Complex hybrid scenarios

Strong Points:
  - Market leader (32% share)
  - Most services and features
  - Largest community
  - Most third-party integrations
  - Best documentation

Watch Out For:
  - Complex pricing
  - IAM complexity
  - Vendor lock-in with proprietary services
```

### Choose GCP When

```yaml
Ideal For:
  - Kubernetes-native workloads
  - Data analytics and ML/AI
  - Cost-sensitive workloads
  - Need for simplicity
  - Multi-region networking

Strong Points:
  - Best Kubernetes experience (GKE)
  - BigQuery for analytics
  - Global network backbone
  - Simpler IAM model
  - Competitive pricing

Watch Out For:
  - Smaller service catalog
  - Fewer enterprise features
  - Less mature support
```

### Choose Azure When

```yaml
Ideal For:
  - Microsoft technology stack
  - Enterprise/Windows workloads
  - Hybrid cloud with on-prem
  - Strong identity requirements (Azure AD)
  - Compliance-heavy industries

Strong Points:
  - Azure AD integration
  - Windows/SQL Server native
  - Azure Arc for hybrid
  - Strong enterprise agreements
  - Government compliance

Watch Out For:
  - Complex pricing model
  - Portal can be overwhelming
  - Some services less mature
```

## 🔄 Multi-Cloud Strategies

### Active-Active Multi-Cloud

```yaml
Architecture:
  ┌─────────────┐     ┌─────────────┐
  │    AWS      │     │    GCP      │
  │  Region 1   │────▶│  Region 2   │
  │  (Primary)  │     │ (Secondary) │
  └─────────────┘     └─────────────┘
         │                   │
         └───────┬───────────┘
                 │
         ┌───────▼───────┐
         │  Global LB    │
         │  (Cloudflare) │
         └───────────────┘

Benefits:
  - No single vendor failure
  - Leverage best-of-breed
  - Negotiating leverage

Challenges:
  - Complexity
  - Data synchronization
  - Skill requirements
  - Cost management
```

### Backup/DR Multi-Cloud

```yaml
Architecture:
  ┌─────────────┐              ┌─────────────┐
  │    AWS      │   Replicate  │   Azure     │
  │  Primary    │─────────────▶│    DR       │
  │             │              │             │
  └─────────────┘              └─────────────┘

Benefits:
  - True disaster recovery
  - Vendor independence
  - Compliance options

Challenges:
  - Replication complexity
  - Different APIs/services
  - Testing difficulty
```

## ✅ Multi-Cloud Best Practices

### Architecture
- [ ] Use cloud-agnostic tools where possible (Terraform, K8s)
- [ ] Abstract cloud-specific services behind interfaces
- [ ] Design for data portability
- [ ] Plan for network connectivity between clouds

### Operations
- [ ] Centralized monitoring (Datadog, Grafana Cloud)
- [ ] Unified identity management
- [ ] Consistent tagging/labeling strategy
- [ ] Cross-cloud cost visibility

### Security
- [ ] Consistent security policies
- [ ] Centralized secrets management
- [ ] Unified audit logging
- [ ] Cross-cloud network security

---

**Next Steps**:
- Learn [AWS Core Services](./aws-core-services.md)
- Explore [Terraform Advanced](./terraform-advanced.md)
- Master [Kubernetes Operations](./kubernetes-operations.md)


