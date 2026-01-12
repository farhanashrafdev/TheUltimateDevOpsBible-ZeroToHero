# DevSecOps Hands-On Labs

## 🎯 Overview

Practical labs covering security testing, secrets management, and secure CI/CD pipelines.

## 📚 Lab 1: SAST/SCA Pipeline

**Objective**: Build a security scanning pipeline with Semgrep and Trivy

### Setup

```bash
# Clone sample vulnerable app
git clone https://github.com/OWASP/WebGoat
cd WebGoat

# Create GitHub Actions workflow
mkdir -p .github/workflows
```

### Create Pipeline

```yaml
# .github/workflows/security-scan.yml
name: Security Scan

on: [push, pull_request]

jobs:
  sast:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Semgrep SAST
        uses: returntocorp/semgrep-action@v1
        with:
          config: >-
            p/security-audit
            p/owasp-top-ten
          generateSarif: true
          
      - name: Upload SARIF
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: semgrep.sarif

  sca:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Trivy SCA
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          severity: 'CRITICAL,HIGH'
```

### Verification

1. Push code and check Actions tab
2. Review Security tab for findings
3. Fix one vulnerability and re-run

---

## 📚 Lab 2: Secrets Management with Vault

**Objective**: Set up HashiCorp Vault and integrate with Kubernetes

### Setup Vault

```bash
# Start Vault in dev mode
docker run -d --name vault -p 8200:8200 \
  -e 'VAULT_DEV_ROOT_TOKEN_ID=root' \
  hashicorp/vault

# Configure
export VAULT_ADDR='http://localhost:8200'
export VAULT_TOKEN='root'

# Enable KV secrets
vault secrets enable -path=secret kv-v2

# Create secret
vault kv put secret/myapp/db \
  username=appuser \
  password=secretpass123
```

### Access from Application

```python
import hvac

client = hvac.Client(url='http://localhost:8200', token='root')
secret = client.secrets.kv.v2.read_secret_version(path='myapp/db')

db_user = secret['data']['data']['username']
db_pass = secret['data']['data']['password']
```

---

## 📚 Lab 3: Container Image Signing

**Objective**: Sign and verify container images with Cosign

### Generate Keys

```bash
# Install cosign
brew install cosign

# Generate key pair
cosign generate-key-pair
```

### Sign and Verify

```bash
# Build and push image
docker build -t ghcr.io/myorg/app:v1 .
docker push ghcr.io/myorg/app:v1

# Sign
cosign sign --key cosign.key ghcr.io/myorg/app:v1

# Verify
cosign verify --key cosign.pub ghcr.io/myorg/app:v1
```

---

## 📚 Lab 4: Kubernetes Admission Control

**Objective**: Enforce security policies with Kyverno

### Install Kyverno

```bash
helm repo add kyverno https://kyverno.github.io/kyverno/
helm install kyverno kyverno/kyverno -n kyverno --create-namespace
```

### Create Policy

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-labels
spec:
  validationFailureAction: Enforce
  rules:
    - name: check-team-label
      match:
        any:
          - resources:
              kinds:
                - Pod
      validate:
        message: "Label 'team' is required"
        pattern:
          metadata:
            labels:
              team: "?*"
```

### Test

```bash
# Should fail
kubectl run test --image=nginx

# Should succeed
kubectl run test --image=nginx --labels="team=platform"
```

---

## ✅ Completion Checklist

- [ ] Lab 1: SAST/SCA Pipeline
- [ ] Lab 2: Vault secrets management
- [ ] Lab 3: Image signing
- [ ] Lab 4: Admission control

