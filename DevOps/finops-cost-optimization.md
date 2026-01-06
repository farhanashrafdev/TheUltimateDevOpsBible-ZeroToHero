# FinOps & Cloud Cost Optimization: Complete Guide

## 🎯 Introduction

FinOps (Financial Operations) is the practice of bringing financial accountability to cloud spending. It combines systems, best practices, and culture to enable organizations to get maximum business value from their cloud investments.

### Why FinOps Matters

```
Cloud Spending Reality:
├── 30% of cloud spend is wasted (industry average)
├── Costs can grow 10x faster than revenue
├── Shadow IT creates hidden expenses
└── Engineers often don't know what things cost

FinOps Goals:
├── Visibility into cloud spending
├── Accountability for costs
├── Optimization without sacrificing performance
└── Data-driven decisions
```

## 📚 FinOps Framework

### The Three Phases

```
┌─────────────────────────────────────────────────────────────┐
│                   FinOps Lifecycle                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────┐                                           │
│  │   INFORM     │  ← Visibility & Allocation                │
│  │              │                                           │
│  │ • Tagging    │                                           │
│  │ • Reporting  │                                           │
│  │ • Showback   │                                           │
│  └──────┬───────┘                                           │
│         │                                                    │
│         ▼                                                    │
│  ┌──────────────┐                                           │
│  │   OPTIMIZE   │  ← Rates & Usage                          │
│  │              │                                           │
│  │ • Right-size │                                           │
│  │ • Reserved   │                                           │
│  │ • Spot/Preempt│                                          │
│  └──────┬───────┘                                           │
│         │                                                    │
│         ▼                                                    │
│  ┌──────────────┐                                           │
│  │   OPERATE    │  ← Continuous Improvement                 │
│  │              │                                           │
│  │ • Governance │                                           │
│  │ • Automation │                                           │
│  │ • Culture    │                                           │
│  └──────────────┘                                           │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## 💰 AWS Cost Optimization

### 1. EC2 Optimization

#### Right-Sizing

```bash
# Find underutilized instances using AWS CLI
aws ce get-rightsizing-recommendation \
  --service "AmazonEC2" \
  --configuration '{
    "RecommendationTarget": "SAME_INSTANCE_FAMILY",
    "BenefitsConsidered": true
  }'
```

```yaml
Right-Sizing Metrics:
  CPU Utilization:
    - < 40% average: Consider downsizing
    - > 80% average: Consider upsizing
  
  Memory Utilization:
    - Monitor with CloudWatch Agent
    - Right-size based on actual usage
  
  Network:
    - Match instance type to bandwidth needs
```

#### Reserved Instances vs Savings Plans

```yaml
Reserved Instances (RI):
  Standard RI:
    - 1 or 3 year commitment
    - Up to 72% discount
    - Specific instance type and region
    - Less flexible
  
  Convertible RI:
    - Can change instance family
    - Up to 54% discount
    - More flexibility

Savings Plans:
  Compute Savings Plans:
    - Apply to EC2, Lambda, Fargate
    - Up to 66% discount
    - Most flexible
  
  EC2 Instance Savings Plans:
    - Specific instance family
    - Up to 72% discount
    - Regional flexibility

Recommendation:
  - New workloads: Start with Savings Plans
  - Stable workloads: Consider Standard RI
  - Variable workloads: Convertible RI or Savings Plans
```

#### Spot Instances

```yaml
# Kubernetes Spot Instance Configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: aws-node-termination-handler-config
data:
  DELETE_LOCAL_DATA: "true"
  ENABLE_SPOT_INTERRUPTION_DRAINING: "true"
---
# Karpenter Spot Configuration
apiVersion: karpenter.sh/v1alpha5
kind: Provisioner
metadata:
  name: spot-provisioner
spec:
  requirements:
    - key: karpenter.sh/capacity-type
      operator: In
      values: ["spot"]
    - key: node.kubernetes.io/instance-type
      operator: In
      values: ["m5.large", "m5.xlarge", "m5a.large", "m5a.xlarge"]
  limits:
    resources:
      cpu: 1000
  ttlSecondsAfterEmpty: 30
```

```bash
# Check Spot pricing
aws ec2 describe-spot-price-history \
  --instance-types m5.large \
  --start-time $(date -u +"%Y-%m-%dT%H:%M:%SZ") \
  --product-descriptions "Linux/UNIX"
```

**Spot Best Practices:**
- Use multiple instance types (flexibility)
- Implement graceful shutdown handling
- Use Spot Fleet for diversification
- Set up Spot interruption handling

### 2. Storage Optimization

#### S3 Lifecycle Policies

```json
{
  "Rules": [
    {
      "ID": "TransitionToIntelligentTiering",
      "Status": "Enabled",
      "Filter": {
        "Prefix": "data/"
      },
      "Transitions": [
        {
          "Days": 0,
          "StorageClass": "INTELLIGENT_TIERING"
        }
      ]
    },
    {
      "ID": "ArchiveOldData",
      "Status": "Enabled",
      "Filter": {
        "Prefix": "logs/"
      },
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "STANDARD_IA"
        },
        {
          "Days": 90,
          "StorageClass": "GLACIER"
        },
        {
          "Days": 365,
          "StorageClass": "DEEP_ARCHIVE"
        }
      ],
      "Expiration": {
        "Days": 730
      }
    }
  ]
}
```

```yaml
S3 Storage Classes (per GB/month):
  Standard:           $0.023
  Intelligent-Tiering: $0.023 (auto-optimizes)
  Standard-IA:        $0.0125
  One Zone-IA:        $0.01
  Glacier Instant:    $0.004
  Glacier Flexible:   $0.0036
  Glacier Deep:       $0.00099

Savings Example (1 TB logs):
  Standard:      $23.55/month
  With Lifecycle: $4.00/month (83% savings)
```

#### EBS Optimization

```bash
# Find unattached EBS volumes
aws ec2 describe-volumes \
  --filters "Name=status,Values=available" \
  --query 'Volumes[*].[VolumeId,Size,VolumeType,CreateTime]'

# Find volumes with low IOPS usage
aws cloudwatch get-metric-statistics \
  --namespace AWS/EBS \
  --metric-name VolumeReadOps \
  --dimensions Name=VolumeId,Value=vol-xxx \
  --start-time $(date -d '7 days ago' -u +%Y-%m-%dT%H:%M:%SZ) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%SZ) \
  --period 86400 \
  --statistics Average
```

```yaml
EBS Optimization Actions:
  gp2 to gp3:
    - Same performance, 20% cheaper
    - No downtime migration
  
  Delete Snapshots:
    - Review old snapshots
    - Implement lifecycle policies
  
  Resize Volumes:
    - Monitor actual usage
    - Decrease oversized volumes
```

### 3. Data Transfer Optimization

```yaml
Data Transfer Costs:
  Within AZ:         Free
  Cross-AZ:          $0.01/GB each way
  Cross-Region:      $0.02/GB
  To Internet:       $0.09/GB (first 10TB)

Optimization Strategies:
  - Use VPC Endpoints for AWS services
  - Keep traffic within same AZ when possible
  - Use CloudFront for content delivery
  - Compress data before transfer
```

### 4. AWS Cost Tools

#### AWS Cost Explorer

```bash
# Get cost and usage report
aws ce get-cost-and-usage \
  --time-period Start=2024-01-01,End=2024-01-31 \
  --granularity MONTHLY \
  --metrics "BlendedCost" "UnblendedCost" "UsageQuantity" \
  --group-by Type=DIMENSION,Key=SERVICE
```

#### AWS Budgets

```bash
# Create budget alert
aws budgets create-budget \
  --account-id 123456789012 \
  --budget '{
    "BudgetName": "Monthly-Cost-Budget",
    "BudgetLimit": {
      "Amount": "1000",
      "Unit": "USD"
    },
    "BudgetType": "COST",
    "TimeUnit": "MONTHLY"
  }' \
  --notifications-with-subscribers '[
    {
      "Notification": {
        "NotificationType": "ACTUAL",
        "ComparisonOperator": "GREATER_THAN",
        "Threshold": 80
      },
      "Subscribers": [
        {
          "SubscriptionType": "EMAIL",
          "Address": "team@example.com"
        }
      ]
    }
  ]'
```

## ☸️ Kubernetes Cost Optimization

### 1. Resource Requests and Limits

```yaml
# Pod with proper resource configuration
apiVersion: v1
kind: Pod
metadata:
  name: optimized-pod
spec:
  containers:
  - name: app
    image: myapp:latest
    resources:
      requests:
        memory: "256Mi"    # Guaranteed memory
        cpu: "250m"        # 0.25 CPU cores
      limits:
        memory: "512Mi"    # Max memory
        cpu: "500m"        # Max CPU
```

```yaml
Resource Optimization Tips:
  Requests:
    - Set based on P50 usage + buffer
    - Affects scheduling decisions
    - Under-requesting = OOM kills
  
  Limits:
    - Set based on P99 usage
    - Prevents noisy neighbors
    - Over-limiting = throttling
  
  Best Practice:
    - Monitor actual usage with metrics-server
    - Use VPA for recommendations
    - Review quarterly
```

### 2. Vertical Pod Autoscaler (VPA)

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: my-app
  updatePolicy:
    updateMode: "Auto"  # Or "Off" for recommendations only
  resourcePolicy:
    containerPolicies:
    - containerName: "*"
      minAllowed:
        cpu: 100m
        memory: 128Mi
      maxAllowed:
        cpu: 2
        memory: 4Gi
```

### 3. Cluster Autoscaler

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
spec:
  template:
    spec:
      containers:
      - name: cluster-autoscaler
        image: k8s.gcr.io/autoscaling/cluster-autoscaler:v1.26.0
        command:
        - ./cluster-autoscaler
        - --cloud-provider=aws
        - --namespace=kube-system
        - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/my-cluster
        - --scale-down-enabled=true
        - --scale-down-delay-after-add=10m
        - --scale-down-unneeded-time=10m
        - --scale-down-utilization-threshold=0.5  # Scale down if < 50% utilized
```

### 4. Kubecost (Cost Monitoring)

```bash
# Install Kubecost
helm repo add kubecost https://kubecost.github.io/cost-analyzer/
helm install kubecost kubecost/cost-analyzer \
  --namespace kubecost --create-namespace \
  --set kubecostToken="your-token"
```

```yaml
Kubecost Features:
  - Real-time cost allocation
  - Cost by namespace/deployment/pod
  - Efficiency recommendations
  - Budget alerts
  - Showback reports
```

### 5. Namespace Resource Quotas

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-quota
  namespace: team-a
spec:
  hard:
    requests.cpu: "10"
    requests.memory: 20Gi
    limits.cpu: "20"
    limits.memory: 40Gi
    pods: "50"
    services: "10"
    persistentvolumeclaims: "10"
```

## 📊 Cost Allocation & Tagging

### Tagging Strategy

```yaml
Required Tags:
  Environment:
    Values: [production, staging, development, sandbox]
    Purpose: Separate costs by environment
  
  Team:
    Values: [platform, backend, frontend, data, security]
    Purpose: Allocate to cost centers
  
  Project:
    Values: [project-a, project-b, shared-services]
    Purpose: Track project spending
  
  Owner:
    Values: [email addresses]
    Purpose: Accountability

Optional Tags:
  CostCenter: Financial allocation
  Application: Specific application
  CreatedBy: Automation tracking
  ExpirationDate: Temporary resources
```

### AWS Tag Policy

```json
{
  "tags": {
    "Environment": {
      "tag_key": {
        "@@assign": "Environment"
      },
      "tag_value": {
        "@@assign": ["production", "staging", "development"]
      },
      "enforced_for": {
        "@@assign": ["ec2:instance", "rds:db", "s3:bucket"]
      }
    }
  }
}
```

### Terraform Tagging

```hcl
# Default tags for all resources
provider "aws" {
  default_tags {
    tags = {
      Environment = var.environment
      Team        = var.team
      Project     = var.project
      ManagedBy   = "terraform"
    }
  }
}

# Module with required tags
module "ec2" {
  source = "./modules/ec2"
  
  tags = merge(local.common_tags, {
    Name = "web-server"
    Role = "application"
  })
}
```

## 🔧 Automation & Governance

### Scheduled Scaling

```yaml
# Kubernetes CronJob for scaling
apiVersion: batch/v1
kind: CronJob
metadata:
  name: scale-down-night
spec:
  schedule: "0 20 * * 1-5"  # 8 PM weekdays
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: kubectl
            image: bitnami/kubectl
            command:
            - /bin/sh
            - -c
            - kubectl scale deployment -n development --all --replicas=0
          restartPolicy: OnFailure
---
apiVersion: batch/v1
kind: CronJob
metadata:
  name: scale-up-morning
spec:
  schedule: "0 8 * * 1-5"  # 8 AM weekdays
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: kubectl
            image: bitnami/kubectl
            command:
            - /bin/sh
            - -c
            - kubectl scale deployment -n development --all --replicas=2
          restartPolicy: OnFailure
```

### Terraform Cost Estimation

```bash
# Using Infracost
infracost breakdown --path .

# In CI/CD pipeline
infracost diff --path . \
  --compare-to infracost-base.json \
  --format json > infracost-diff.json

# Comment on PR with cost change
infracost comment github --path infracost-diff.json \
  --repo $GITHUB_REPOSITORY \
  --pull-request $PR_NUMBER \
  --github-token $GITHUB_TOKEN
```

### AWS Lambda for Cost Automation

```python
import boto3
from datetime import datetime

def lambda_handler(event, context):
    """Stop non-production EC2 instances outside business hours."""
    ec2 = boto3.client('ec2')
    
    # Find instances tagged as non-production
    response = ec2.describe_instances(
        Filters=[
            {'Name': 'tag:Environment', 'Values': ['development', 'staging']},
            {'Name': 'instance-state-name', 'Values': ['running']}
        ]
    )
    
    instance_ids = []
    for reservation in response['Reservations']:
        for instance in reservation['Instances']:
            instance_ids.append(instance['InstanceId'])
    
    if instance_ids:
        ec2.stop_instances(InstanceIds=instance_ids)
        print(f"Stopped instances: {instance_ids}")
    
    return {'statusCode': 200, 'body': f'Stopped {len(instance_ids)} instances'}
```

## 📈 Cost Monitoring Dashboard

### Grafana Dashboard Queries

```yaml
# Prometheus queries for cost metrics
panels:
  - title: "Cluster Cost (Daily)"
    query: |
      sum(
        container_memory_usage_bytes{namespace!="kube-system"} 
        * on(node) group_left() 
        node_ram_hourly_cost
      ) * 24

  - title: "Cost by Namespace"
    query: |
      sum by (namespace) (
        container_memory_usage_bytes 
        * on(node) group_left() 
        node_ram_hourly_cost
      )

  - title: "Idle Resources"
    query: |
      1 - (
        sum(rate(container_cpu_usage_seconds_total[5m])) 
        / 
        sum(kube_pod_container_resource_requests{resource="cpu"})
      )
```

### Cost Alerts

```yaml
# Prometheus AlertManager rules
groups:
- name: cost-alerts
  rules:
  - alert: HighNamespaceCost
    expr: |
      sum by (namespace) (
        container_memory_usage_bytes * 0.000001 * 0.05
      ) > 100
    for: 1h
    labels:
      severity: warning
    annotations:
      summary: "Namespace {{ $labels.namespace }} cost exceeds $100/day"
      
  - alert: UnusedPersistentVolume
    expr: |
      kube_persistentvolume_status_phase{phase="Available"} == 1
    for: 24h
    labels:
      severity: info
    annotations:
      summary: "PV {{ $labels.persistentvolume }} has been unbound for 24h"
```

## ✅ Cost Optimization Checklist

### Quick Wins (Week 1)
- [ ] Enable AWS Cost Explorer
- [ ] Set up budget alerts
- [ ] Identify and delete unused resources
- [ ] Implement tagging strategy
- [ ] Review and terminate unattached EBS volumes

### Medium-Term (Month 1)
- [ ] Right-size EC2 instances
- [ ] Implement S3 lifecycle policies
- [ ] Purchase Savings Plans for steady-state workloads
- [ ] Set up Kubecost or similar
- [ ] Implement scheduled scaling for dev/staging

### Long-Term (Quarter 1)
- [ ] Build cost allocation reports
- [ ] Implement chargeback/showback
- [ ] Automate cost governance
- [ ] Train teams on cost awareness
- [ ] Establish FinOps culture

## 📊 Key Metrics to Track

```yaml
Financial Metrics:
  - Total cloud spend (monthly)
  - Cost per customer/transaction
  - Cost variance (actual vs budget)
  - Reserved instance utilization
  - Savings plan utilization

Efficiency Metrics:
  - Resource utilization (CPU, memory)
  - Idle resource percentage
  - Cost per deployment
  - Unit economics (cost per API call, etc.)

Coverage Metrics:
  - Tagging compliance percentage
  - Commitment coverage (RI/SP)
  - Spot instance percentage
```

---

**Next Steps**:
- Learn [Monitoring & Observability](./monitoring-observability.md)
- Explore [AWS Core Services](./aws-core-services.md)
- Master [Kubernetes Operations](./kubernetes-operations.md)

**Remember**: FinOps is a cultural shift, not just a tool implementation. Start with visibility, build accountability, then optimize continuously. Every engineer should understand the cost impact of their decisions.


