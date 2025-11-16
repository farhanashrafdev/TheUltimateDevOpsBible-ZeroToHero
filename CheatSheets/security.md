# Security Cheat Sheet - Complete Reference

## 🔍 Container Scanning

### Trivy

```bash
# Scan container image
trivy image nginx:latest
trivy image --severity HIGH,CRITICAL nginx:latest
trivy image --format json nginx:latest
trivy image --exit-code 1 nginx:latest  # Fail on vulnerabilities

# Scan filesystem
trivy fs .
trivy fs --severity HIGH,CRITICAL .
trivy fs --scanners vuln,secret,config .

# Scan Kubernetes
trivy k8s cluster
trivy k8s cluster --severity HIGH,CRITICAL
trivy k8s cluster --format json

# Scan repository
trivy repo https://github.com/user/repo
trivy repo --severity HIGH,CRITICAL https://github.com/user/repo

# Scan with config
trivy image --config-policy policies/ nginx:latest
```

### Snyk

```bash
# Test dependencies
snyk test
snyk test --severity-threshold=high
snyk test --json

# Monitor project
snyk monitor
snyk monitor --org=myorg

# Test container
snyk container test nginx:latest
snyk container test nginx:latest --severity-threshold=high

# Test infrastructure
snyk iac test
snyk iac test terraform/
snyk iac test --severity-threshold=high

# Fix vulnerabilities
snyk fix
```

### Clair

```bash
# Scan with Clair
clair-scanner --ip host nginx:latest
clair-scanner --report report.json nginx:latest
```

## 🏗️ Infrastructure Scanning

### Checkov

```bash
# Scan Terraform
checkov -d terraform/
checkov -d terraform/ --framework terraform
checkov -d terraform/ --output json
checkov -d terraform/ --soft-fail

# Scan CloudFormation
checkov -f template.yaml --framework cloudformation

# Scan Kubernetes
checkov -f deployment.yaml --framework kubernetes

# Scan Dockerfile
checkov -f Dockerfile --framework dockerfile

# Scan with policy
checkov -d . --check CKV_AWS_20
checkov -d . --skip-check CKV_AWS_20
```

### Terrascan

```bash
# Scan infrastructure
terrascan scan
terrascan scan -t terraform
terrascan scan -t k8s
terrascan scan -t docker

# Scan with policy
terrascan scan -p policies/
terrascan scan --policy-type all
```

### tfsec

```bash
# Scan Terraform
tfsec .
tfsec --format json
tfsec --minimum-severity HIGH
tfsec --exclude-check AWS001
```

## 🔐 Secrets Management

### HashiCorp Vault

```bash
# Authenticate
vault auth -method=userpass username=user
vault login -method=userpass username=user

# Read secret
vault kv get secret/app
vault kv get -field=password secret/app

# Write secret
vault kv put secret/app username=admin password=secret123
vault kv put secret/app @secret.json

# List secrets
vault kv list secret/

# Delete secret
vault kv delete secret/app

# Policy
vault policy write mypolicy policy.hcl
vault policy read mypolicy
```

### AWS Secrets Manager

```bash
# Get secret
aws secretsmanager get-secret-value --secret-id mysecret
aws secretsmanager get-secret-value --secret-id mysecret --query SecretString --output text

# Create secret
aws secretsmanager create-secret --name mysecret --secret-string '{"key":"value"}'

# Update secret
aws secretsmanager update-secret --secret-id mysecret --secret-string '{"key":"newvalue"}'

# List secrets
aws secretsmanager list-secrets
```

### SOPS

```bash
# Encrypt file
sops -e secrets.yaml > secrets.enc.yaml

# Decrypt file
sops -d secrets.enc.yaml

# Edit encrypted file
sops secrets.enc.yaml

# Encrypt with AWS KMS
sops -e -k arn:aws:kms:region:account:key secrets.yaml
```

## 🔒 SAST/SCA/DAST

### SAST Tools

```bash
# Semgrep
semgrep --config=auto .
semgrep --config=p/r2c-security-audit .

# Bandit (Python)
bandit -r .
bandit -r . -f json

# ESLint Security
eslint --plugin security .
```

### SCA Tools

```bash
# OWASP Dependency Check
dependency-check.sh --project myproject --scan .

# npm audit
npm audit
npm audit fix

# pip-audit
pip-audit
```

### DAST Tools

```bash
# OWASP ZAP
zap-baseline.py -t http://target
zap-full-scan.py -t http://target

# Nikto
nikto -h http://target
```

## 🛡️ Kubernetes Security

### Pod Security

```yaml
# Security context
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  allowPrivilegeEscalation: false
  capabilities:
    drop:
      - ALL
    add:
      - NET_BIND_SERVICE
  readOnlyRootFilesystem: true
```

### Network Policies

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

### RBAC

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```

## 🔍 Security Best Practices

### Container Security

```dockerfile
# Use non-root user
RUN adduser -D appuser
USER appuser

# Scan base image
# Use minimal base images
FROM alpine:latest

# Don't store secrets
# Use secrets management
```

### Infrastructure Security

```hcl
# Enable encryption
resource "aws_s3_bucket" "data" {
  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm = "AES256"
      }
    }
  }
}
```

---

**Remember**: Security is a continuous process. Scan regularly, rotate secrets, follow least privilege, and keep dependencies updated.
