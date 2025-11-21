# Pipeline Scanning: Integrating Security into CI/CD

## 🎯 Introduction

Integrate security scanning at every stage of the pipeline.

## 🔍 Scanning Stages

### 1. Code Commit

**Secret Scanning**:
```yaml
- name: Secret scan
  run: |
    trufflehog git file://. --only-verified
    gitleaks detect --source .
```

### 2. Build

**SAST (Static Application Security Testing)**:
```yaml
- name: SAST
  run: |
    semgrep --config=auto --error
    sonar-scanner
```

**SCA (Software Composition Analysis)**:
```yaml
- name: Dependency scan
  run: |
    npm audit --audit-level=high
    snyk test
```

### 3. Container Build

**Image Scanning**:
```yaml
- name: Build and scan
  run: |
    docker build -t myapp:${{ github.sha }} .
    trivy image --severity HIGH,CRITICAL myapp:${{ github.sha }}
```

### 4. Pre-Deployment

**IaC Scanning**:
```yaml
- name: IaC scan
  run: |
    checkov -d kubernetes/
    tfsec terraform/
```

---

## 🎯 Complete Pipeline

```yaml
name: Secure Pipeline

on: [push]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Secret scan
        run: gitleaks detect
        
      - name: SAST
        run: semgrep --config=auto
        
      - name: SCA
        run: snyk test
        
      - name: Build
        run: docker build -t myapp .
        
      - name: Container scan
        run: trivy image myapp
        
      - name: IaC scan
        run: checkov -d k8s/
```

---

## ✅ Best Practices

- [ ] Scan at every stage
- [ ] Fail builds on high/critical issues
- [ ] Generate security reports
- [ ] Track metrics

---

**Next**: [Secrets Management](./secrets-management.md).
