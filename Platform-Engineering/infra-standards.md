# Infrastructure Standards

## 🎯 Introduction

Infrastructure standards ensure consistency, security, and maintainability across all infrastructure components.

## 📚 Standard Categories

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Infrastructure Standards                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Naming Conventions        Tagging Policy         Security          │
│  ├── {env}-{app}-{type}   ├── Environment        ├── Encryption    │
│  ├── lowercase             ├── Team              ├── Network rules │
│  └── No underscores        ├── Cost-center       └── IAM policies  │
│                            └── Expiration                           │
│                                                                      │
│  Resource Sizing           Networking            Monitoring         │
│  ├── Environment limits    ├── VPC design        ├── Required      │
│  ├── Right-sizing          ├── Subnet layout         metrics       │
│  └── Cost controls         └── Security groups   ├── Alerting      │
│                                                   └── Dashboards    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 🔧 Terraform Module Standards

### Module Structure

```
modules/
├── aws-vpc/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   ├── versions.tf
│   ├── README.md
│   └── examples/
│       └── basic/
└── aws-eks/
    └── ...
```

### Required Variables

```hcl
variable "environment" {
  type        = string
  description = "Environment name (dev, staging, prod)"
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "tags" {
  type = object({
    team        = string
    cost_center = string
    project     = string
  })
  description = "Required resource tags"
}
```

### Policy Enforcement (OPA)

```rego
# policy/terraform.rego
package terraform

deny[msg] {
  resource := input.resource_changes[_]
  resource.type == "aws_s3_bucket"
  not resource.change.after.tags.team
  msg := sprintf("S3 bucket %v missing 'team' tag", [resource.name])
}

deny[msg] {
  resource := input.resource_changes[_]
  resource.type == "aws_instance"
  resource.change.after.instance_type == "m5.24xlarge"
  msg := "Instance type m5.24xlarge requires approval"
}
```

## 📝 Kubernetes Standards

### Pod Security

```yaml
# Required in all deployments
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  readOnlyRootFilesystem: true
  allowPrivilegeEscalation: false
  capabilities:
    drop: ["ALL"]
```

### Resource Requirements

```yaml
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
  limits:
    memory: "256Mi"
    cpu: "500m"
```

### Required Labels

```yaml
metadata:
  labels:
    app.kubernetes.io/name: myapp
    app.kubernetes.io/component: backend
    team: platform
    environment: production
```

## ✅ Enforcement

1. **Pre-commit**: Terraform fmt/validate
2. **CI Pipeline**: OPA/Conftest policies
3. **Admission Control**: Kyverno/OPA Gatekeeper
4. **Drift Detection**: Regular compliance scans

---

**Next**: Return to [Platform Engineering Overview](./idp-overview.md).

