# Infrastructure as Code Scanning

## 🎯 Securing Infrastructure Code

Scan Infrastructure as Code (Terraform, CloudFormation, etc.) for security misconfigurations and policy violations.

## 🔍 Scanning Tools

### Checkov
- Terraform, CloudFormation, Kubernetes
- Policy as code
- Custom policies

### Terrascan
- Multi-cloud support
- Policy library
- CI/CD integration

### tfsec
- Terraform focused
- Fast scanning
- Comprehensive rules

## 📝 Examples

### Checkov
```bash
checkov -d terraform/
checkov -f main.tf --framework terraform
```

### Terrascan
```bash
terrascan scan -t terraform
terrascan scan -t k8s
```

## 📋 Common Checks

- Public S3 buckets
- Unencrypted storage
- Open security groups
- Missing IAM policies
- Hardcoded secrets

## ✅ Best Practices

- Scan before apply
- Use in CI/CD
- Custom policies
- Regular updates
- Fix misconfigurations

---

**Next**: Learn secrets management.

