# CI/CD Security: Securing the Pipeline

## 🎯 Introduction

Your CI/CD pipeline has access to **everything**: source code, secrets, production infrastructure. A compromised pipeline = compromised production. This guide covers how to secure your entire CI/CD workflow.

## 🔐 Core Principles

1. **Least Privilege**: Pipelines should have minimum necessary permissions
2. **Secrets Management**: Never hardcode secrets
3. **Artifact Integrity**: Sign and verify all artifacts
4. **Audit Everything**: Log all pipeline activities
5. **Fail Securely**: Security checks should fail the build

---

## 🛡️ Pipeline Security Layers

### 1. Source Code Security

**Branch Protection**:
```yaml
# GitHub branch protection rules
- Require pull request reviews (2+ approvers)
- Require status checks to pass
- Require signed commits
- Restrict who can push
- Require linear history
```

**Signed Commits**:
```bash
# Configure GPG signing
git config --global user.signingkey YOUR_GPG_KEY
git config --global commit.gpgsign true

# Verify commits
git log --show-signature
```

### 2. Build Security

**Secure Build Environment**:
- Use ephemeral build agents (destroyed after each build)
- Run builds in isolated containers
- Scan build dependencies
- Validate build inputs

**Example: GitHub Actions**:
```yaml
name: Secure Build

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read      # Read-only access to code
      packages: write     # Write to package registry
      
    steps:
      - uses: actions/checkout@v3
      
      - name: Scan dependencies
        run: |
          npm audit
          snyk test
          
      - name: Build
        run: npm run build
        
      - name: Sign artifact
        run: cosign sign artifact.tar.gz
```

### 3. Secrets Management

**Never Do This**:
```yaml
# ❌ BAD: Hardcoded secrets
env:
  DATABASE_PASSWORD: "supersecret123"
  API_KEY: "abc123xyz"
```

**Do This Instead**:
```yaml
# ✅ GOOD: Use secrets management
env:
  DATABASE_PASSWORD: ${{ secrets.DB_PASSWORD }}
  API_KEY: ${{ secrets.API_KEY }}
```

**Best Practices**:
- Use native secrets management (GitHub Secrets, GitLab CI Variables)
- Rotate secrets regularly
- Use short-lived tokens (OIDC)
- Mask secrets in logs
- Limit secret scope (per environment)

**OIDC for Cloud Access** (No long-lived credentials):
```yaml
# GitHub Actions with AWS OIDC
- name: Configure AWS credentials
  uses: aws-actions/configure-aws-credentials@v2
  with:
    role-to-assume: arn:aws:iam::123456789:role/GitHubActions
    aws-region: us-east-1
```

### 4. Dependency Scanning

**Scan for Vulnerabilities**:
```yaml
- name: Dependency scan
  run: |
    # NPM
    npm audit --audit-level=high
    
    # Python
    safety check
    pip-audit
    
    # Go
    govulncheck ./...
    
    # Multi-language
    snyk test
    trivy fs .
```

### 5. Static Analysis (SAST)

**Scan Code for Security Issues**:
```yaml
- name: SAST scan
  run: |
    # SonarQube
    sonar-scanner
    
    # Semgrep
    semgrep --config=auto
    
    # CodeQL
    codeql analyze
```

### 6. Container Security

**Scan Container Images**:
```yaml
- name: Build image
  run: docker build -t myapp:${{ github.sha }} .
  
- name: Scan image
  run: |
    trivy image myapp:${{ github.sha }}
    grype myapp:${{ github.sha }}
    
- name: Sign image
  run: |
    cosign sign myapp:${{ github.sha }}
```

---

## 🔒 Advanced Security Patterns

### 1. Policy as Code

**Open Policy Agent (OPA)**:
```rego
# policy.rego
package pipeline

deny[msg] {
    input.image.tag == "latest"
    msg = "Using 'latest' tag is not allowed"
}

deny[msg] {
    input.secrets.exposed
    msg = "Secrets detected in logs"
}
```

### 2. Supply Chain Security

**Generate SBOM (Software Bill of Materials)**:
```yaml
- name: Generate SBOM
  run: |
    syft myapp:latest -o spdx-json > sbom.json
    
- name: Attest SBOM
  run: |
    cosign attest --predicate sbom.json myapp:latest
```

**SLSA Provenance**:
```yaml
- name: Generate provenance
  uses: slsa-framework/slsa-github-generator@v1
  with:
    artifact: myapp.tar.gz
```

### 3. Runtime Security

**Falco Rules in CI**:
```yaml
- name: Test with Falco
  run: |
    # Run container with Falco
    docker run --rm \
      -v /var/run/docker.sock:/var/run/docker.sock \
      falcosecurity/falco:latest \
      -r /etc/falco/rules.yaml
```

---

## 🎯 Complete Secure Pipeline Example

```yaml
name: Secure CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:

permissions:
  contents: read
  packages: write
  id-token: write  # For OIDC

jobs:
  security-checks:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Secret scanning
        uses: trufflesecurity/trufflehog@main
        with:
          path: ./
          
      - name: Dependency scan
        run: |
          npm audit --audit-level=high
          snyk test --severity-threshold=high
          
      - name: SAST
        run: semgrep --config=auto --error
        
  build:
    needs: security-checks
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build
        run: npm run build
        
      - name: Build container
        run: docker build -t myapp:${{ github.sha }} .
        
      - name: Scan container
        run: trivy image --severity HIGH,CRITICAL myapp:${{ github.sha }}
        
      - name: Sign image
        run: |
          cosign sign \
            --key ${{ secrets.COSIGN_KEY }} \
            myapp:${{ github.sha }}
            
      - name: Generate SBOM
        run: syft myapp:${{ github.sha }} -o spdx-json > sbom.json
        
      - name: Push image
        run: docker push myapp:${{ github.sha }}
        
  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Configure AWS
        uses: aws-actions/configure-aws-credentials@v2
        with:
          role-to-assume: ${{ secrets.AWS_ROLE }}
          aws-region: us-east-1
          
      - name: Verify image signature
        run: |
          cosign verify \
            --key ${{ secrets.COSIGN_PUBLIC_KEY }} \
            myapp:${{ github.sha }}
            
      - name: Deploy
        run: |
          kubectl set image deployment/myapp \
            myapp=myapp:${{ github.sha }}
```

---

## ✅ Security Checklist

- [ ] Branch protection enabled
- [ ] Signed commits required
- [ ] Secrets in secrets manager (not code)
- [ ] OIDC for cloud access (no long-lived keys)
- [ ] Dependency scanning
- [ ] SAST scanning
- [ ] Container scanning
- [ ] Image signing
- [ ] SBOM generation
- [ ] Policy enforcement
- [ ] Audit logging enabled
- [ ] Least privilege permissions

---

## 📚 Tools

- **Secret Scanning**: TruffleHog, GitGuardian, Gitleaks
- **Dependency Scanning**: Snyk, Dependabot, Renovate
- **SAST**: SonarQube, Semgrep, CodeQL
- **Container Scanning**: Trivy, Grype, Clair
- **Image Signing**: Cosign, Notary
- **SBOM**: Syft, CycloneDX
- **Policy**: OPA, Kyverno

---

**Next**: Learn about [Image Signing](./image-signing.md) and [Supply Chain Security](./supply-chain-security.md).
