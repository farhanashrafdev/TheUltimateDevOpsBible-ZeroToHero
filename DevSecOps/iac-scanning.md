# IaC Scanning: Securing Infrastructure as Code

## 🎯 Introduction

Infrastructure as Code can contain security misconfigurations. Scan IaC **before** deployment to catch issues early.

## 🔍 Tools

### 1. Checkov

**Install**:
```bash
pip install checkov
```

**Scan Terraform**:
```bash
checkov -d terraform/
checkov -f main.tf
```

**Example Issues**:
```hcl
# ❌ BAD: S3 bucket publicly accessible
resource "aws_s3_bucket" "bad" {
  bucket = "my-bucket"
  acl    = "public-read"  # Checkov will flag this
}

# ✅ GOOD: Private bucket
resource "aws_s3_bucket" "good" {
  bucket = "my-bucket"
  acl    = "private"
}
```

### 2. tfsec

**Install**:
```bash
brew install tfsec
```

**Scan**:
```bash
tfsec .
tfsec --format json
```

### 3. Terrascan

**Scan multiple IaC formats**:
```bash
terrascan scan -t terraform
terrascan scan -t kubernetes
```

---

## 🎯 CI/CD Integration

```yaml
name: IaC Security

on: [push, pull_request]

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Checkov
        run: |
          pip install checkov
          checkov -d terraform/ --quiet --compact
          
      - name: tfsec
        uses: aquasecurity/tfsec-action@v1
        with:
          soft_fail: false
```

---

## ✅ Best Practices

- [ ] Scan IaC in CI/CD
- [ ] Fix high/critical issues
- [ ] Use policy as code
- [ ] Scan before deployment

---

**Next**: [Pipeline Scanning](./pipeline-scanning.md) and [KMS/Vault/SOPS](./kms-vault-sops.md).
