# Infrastructure as Code (IaC) Scanning

## 🎯 Introduction

IaC scanning detects security misconfigurations in infrastructure definitions before deployment, catching issues at the cheapest stage to fix.

### Common IaC Security Issues

```
AWS Terraform:                    Kubernetes:
├── S3 buckets with public access ├── Containers running as root
├── Security groups 0.0.0.0/0     ├── Privileged containers
├── Unencrypted EBS volumes       ├── hostNetwork enabled
├── IAM policies with "*"         ├── Missing resource limits
└── RDS without encryption        └── No network policies
```

## 🔧 Terraform Security Scanning

### Checkov

```yaml
# .github/workflows/terraform-security.yml
name: Terraform Security

on:
  push:
    paths: ['**/*.tf']
  pull_request:

jobs:
  checkov:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Checkov Scan
        uses: bridgecrewio/checkov-action@master
        with:
          directory: .
          framework: terraform
          soft_fail: false
          output_format: sarif
          download_external_modules: true
```

#### Checkov Configuration

```yaml
# .checkov.yaml
framework:
  - terraform

skip-check:
  - CKV_AWS_18  # S3 logging handled elsewhere

hard-fail-on:
  - CRITICAL
  - HIGH
```

### tfsec

```yaml
- name: tfsec
  uses: aquasecurity/tfsec-action@v1.0.0
  with:
    soft_fail: false
    format: sarif
```

#### tfsec Configuration

```yaml
# .tfsec.yml
severity_overrides:
  aws-s3-enable-bucket-logging: LOW

minimum_severity: MEDIUM
```

### Terrascan

```yaml
- name: Terrascan
  uses: tenable/terrascan-action@main
  with:
    iac_type: 'terraform'
    iac_version: 'v14'
    policy_type: 'aws'
```

## ☸️ Kubernetes Security Scanning

### Kubesec

```yaml
- name: Kubesec Scan
  uses: controlplaneio/kubesec-action@master
  with:
    input: kubernetes/deployment.yaml
    exit-code: "1"
```

### Kube-linter

```yaml
- name: Kube-linter
  uses: stackrox/kube-linter-action@v1
  with:
    directory: kubernetes/
    config: .kube-linter.yaml
```

**Configuration:**

```yaml
# .kube-linter.yaml
checks:
  addAllBuiltIn: true
  exclude:
    - "unset-cpu-requirements"
    
customChecks:
  - name: "require-team-label"
    template: "required-label"
    params:
      key: "team"
```

### Polaris

```yaml
- name: Polaris Audit
  run: |
    polaris audit \
      --audit-path kubernetes/ \
      --set-exit-code-on-danger \
      --set-exit-code-below-score 80
```

### OPA/Conftest

```rego
# policy/kubernetes.rego
package main

deny[msg] {
  input.kind == "Deployment"
  not input.spec.template.spec.securityContext.runAsNonRoot
  msg = sprintf("Deployment %s must set runAsNonRoot", [input.metadata.name])
}

deny[msg] {
  input.kind == "Deployment"
  container := input.spec.template.spec.containers[_]
  container.securityContext.privileged
  msg = sprintf("Container %s must not be privileged", [container.name])
}

deny[msg] {
  input.kind == "Deployment"
  container := input.spec.template.spec.containers[_]
  not container.resources.limits.memory
  msg = sprintf("Container %s must have memory limits", [container.name])
}
```

## 🐳 Dockerfile Scanning

### Hadolint

```yaml
- name: Hadolint
  uses: hadolint/hadolint-action@v3.1.0
  with:
    dockerfile: Dockerfile
    failure-threshold: error
```

**Configuration:**

```yaml
# .hadolint.yaml
ignored:
  - DL3008  # Pin apt versions

override:
  error:
    - DL3002  # Don't switch to root
  warning:
    - DL3013  # Pin pip packages
```

## 📊 Unified IaC Pipeline

```yaml
name: IaC Security

on:
  push:
    paths: ['**/*.tf', 'kubernetes/**', 'Dockerfile*']

jobs:
  terraform:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: bridgecrewio/checkov-action@master
        with:
          directory: .
          framework: terraform
      - uses: aquasecurity/tfsec-action@v1.0.0

  kubernetes:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: stackrox/kube-linter-action@v1
        with:
          directory: kubernetes/

  dockerfile:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: hadolint/hadolint-action@v3.1.0
        with:
          dockerfile: Dockerfile

  security-gate:
    needs: [terraform, kubernetes, dockerfile]
    if: always()
    runs-on: ubuntu-latest
    steps:
      - name: Check results
        run: |
          if [ "${{ needs.terraform.result }}" == "failure" ]; then
            exit 1
          fi
```

## ✅ Tool Selection

| Use Case | Primary | Secondary |
|----------|---------|-----------|
| Terraform | Checkov | tfsec |
| Kubernetes | Kube-linter | Polaris |
| Dockerfile | Hadolint | Trivy |
| Multi-IaC | Trivy | Checkov |

---

**Next**: Learn about [Image Signing](./image-signing.md) for container verification.

