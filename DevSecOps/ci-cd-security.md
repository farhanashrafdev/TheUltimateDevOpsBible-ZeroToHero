# CI/CD Security

## 🎯 Introduction

CI/CD pipelines are the heart of modern software delivery—and a prime target for attackers. A compromised pipeline can inject malicious code into production, exfiltrate secrets, or provide lateral movement across your infrastructure. This guide covers comprehensive security practices for protecting your CI/CD systems.

### Why CI/CD Security Matters

```
Attack Surface of CI/CD Pipelines:
┌─────────────────────────────────────────────────────────────────────┐
│                         Attack Vectors                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Source Code         Build Process        Deployment                 │
│  ├── Malicious PRs   ├── Dependency       ├── Stolen credentials    │
│  ├── Typosquatting      poisoning         ├── Misconfigured IAM     │
│  ├── Compromised     ├── Build script     ├── Insecure secrets      │
│      dependencies       injection         └── Unauthorized access   │
│  └── Secret leaks    ├── Cache poisoning                            │
│                      └── Runner compromise                           │
│                                                                      │
│  Real-World Incidents:                                               │
│  • SolarWinds (2020): Build system compromise → 18,000+ customers   │
│  • Codecov (2021): CI script modified → secrets exfiltrated         │
│  • ua-parser-js (2021): NPM package hijacked via CI                 │
│  • GitHub Actions (2022): Actions poisoning in popular repos        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 📚 Core Security Principles

### Defense in Depth

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CI/CD Security Layers                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Layer 1: Identity & Access                                          │
│  ├── Strong authentication (SSO, MFA)                               │
│  ├── Role-based access control (RBAC)                               │
│  ├── Least privilege principle                                      │
│  └── Regular access reviews                                         │
│                                                                      │
│  Layer 2: Source Code Protection                                     │
│  ├── Branch protection rules                                        │
│  ├── Required reviews                                               │
│  ├── Signed commits                                                 │
│  └── Pre-commit hooks                                               │
│                                                                      │
│  Layer 3: Pipeline Security                                          │
│  ├── Pinned dependencies                                            │
│  ├── Isolated runners                                               │
│  ├── Secrets management                                             │
│  └── Artifact verification                                          │
│                                                                      │
│  Layer 4: Runtime Protection                                         │
│  ├── Image scanning                                                 │
│  ├── Policy enforcement                                             │
│  ├── Network segmentation                                           │
│  └── Audit logging                                                  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 🔐 Authentication & Authorization

### OIDC (OpenID Connect) for Cloud Authentication

**Why OIDC?**
- No long-lived credentials stored in CI/CD
- Short-lived tokens (typically 1 hour)
- Automatic rotation
- Audit trail of token usage

#### GitHub Actions to AWS

```yaml
# .github/workflows/deploy.yml
name: Deploy to AWS

on:
  push:
    branches: [main]

permissions:
  id-token: write  # Required for OIDC
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/GitHubActionsDeployRole
          role-session-name: github-actions-deploy
          aws-region: us-east-1

      - name: Deploy to S3
        run: aws s3 sync ./dist s3://my-bucket/
```

**AWS IAM Role Trust Policy:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::123456789012:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        },
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:myorg/myrepo:ref:refs/heads/main"
        }
      }
    }
  ]
}
```

#### GitLab CI to AWS

```yaml
# .gitlab-ci.yml
deploy:
  stage: deploy
  image: amazon/aws-cli:latest
  id_tokens:
    AWS_TOKEN:
      aud: https://gitlab.com
  variables:
    ROLE_ARN: arn:aws:iam::123456789012:role/GitLabDeployRole
  before_script:
    - >
      export $(printf "AWS_ACCESS_KEY_ID=%s AWS_SECRET_ACCESS_KEY=%s AWS_SESSION_TOKEN=%s"
      $(aws sts assume-role-with-web-identity
      --role-arn ${ROLE_ARN}
      --role-session-name "GitLabRunner-${CI_PROJECT_ID}-${CI_PIPELINE_ID}"
      --web-identity-token ${AWS_TOKEN}
      --duration-seconds 3600
      --query 'Credentials.[AccessKeyId,SecretAccessKey,SessionToken]'
      --output text))
  script:
    - aws s3 sync ./dist s3://my-bucket/
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
```

#### OIDC to GCP

```yaml
# GitHub Actions to GCP
- name: Authenticate to Google Cloud
  uses: google-github-actions/auth@v2
  with:
    workload_identity_provider: 'projects/123456789/locations/global/workloadIdentityPools/github-pool/providers/github-provider'
    service_account: 'github-actions@my-project.iam.gserviceaccount.com'

- name: Deploy to Cloud Run
  run: |
    gcloud run deploy my-service \
      --image gcr.io/my-project/my-image:${{ github.sha }} \
      --region us-central1
```

### Service Account Best Practices

```yaml
# Bad: Overly permissive
service_account:
  name: ci-cd-admin
  permissions:
    - "*:*"  # Never do this!

# Good: Least privilege
service_account:
  name: ci-cd-deployer
  permissions:
    - s3:PutObject
    - s3:GetObject
    - ecr:GetAuthorizationToken
    - ecr:BatchCheckLayerAvailability
    - ecr:PutImage
  conditions:
    - resource: "arn:aws:s3:::production-bucket/*"
    - source_ip: ["10.0.0.0/8"]  # Only from CI runners
```

## 🔑 Secrets Management

### Secrets Hierarchy

```
┌─────────────────────────────────────────────────────────────────────┐
│                     Secrets Management Hierarchy                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Level 1: CI Platform Secrets (GitHub Secrets, GitLab Variables)    │
│  ├── Pros: Easy to use, integrated                                  │
│  ├── Cons: Limited rotation, platform-locked                        │
│  └── Use for: Non-critical, platform-specific secrets               │
│                                                                      │
│  Level 2: External Secret Managers (Vault, AWS Secrets Manager)     │
│  ├── Pros: Centralized, rotation, audit logs, dynamic secrets       │
│  ├── Cons: Additional infrastructure                                │
│  └── Use for: Production credentials, database passwords            │
│                                                                      │
│  Level 3: OIDC / Workload Identity (No secrets at all!)             │
│  ├── Pros: No secrets to manage, automatic rotation                 │
│  ├── Cons: Requires cloud provider support                          │
│  └── Use for: Cloud provider access                                 │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### GitHub Actions Secrets

```yaml
# .github/workflows/secure-deploy.yml
name: Secure Deployment

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production  # Requires approval
    steps:
      - uses: actions/checkout@v4
      
      # Reference secrets securely
      - name: Deploy
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
          API_KEY: ${{ secrets.API_KEY }}
        run: |
          # Never echo secrets!
          # echo $DATABASE_URL  # BAD!
          
          # Use secrets directly in commands
          ./deploy.sh
```

### HashiCorp Vault Integration

```yaml
# GitHub Actions with Vault
- name: Import Secrets from Vault
  uses: hashicorp/vault-action@v2
  with:
    url: https://vault.company.com
    method: jwt
    role: github-actions
    secrets: |
      secret/data/production/db username | DB_USERNAME ;
      secret/data/production/db password | DB_PASSWORD ;
      secret/data/production/api key | API_KEY

- name: Use secrets
  run: |
    # Secrets are available as environment variables
    ./configure-app.sh
```

**Vault Policy for CI/CD:**

```hcl
# vault-policy.hcl
path "secret/data/production/*" {
  capabilities = ["read"]
}

path "secret/data/staging/*" {
  capabilities = ["read"]
}

# Deny access to admin paths
path "secret/data/admin/*" {
  capabilities = ["deny"]
}

# Allow token self-lookup
path "auth/token/lookup-self" {
  capabilities = ["read"]
}
```

### Secret Scanning Prevention

```yaml
# .github/workflows/secret-scan.yml
name: Secret Scanning

on:
  pull_request:
  push:

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for scanning

      - name: TruffleHog Scan
        uses: trufflesecurity/trufflehog@main
        with:
          path: ./
          base: ${{ github.event.repository.default_branch }}
          head: HEAD
          extra_args: --only-verified

      - name: Gitleaks Scan
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

**.gitleaks.toml Configuration:**

```toml
[extend]
useDefault = true

[[rules]]
id = "custom-api-key"
description = "Custom API Key Pattern"
regex = '''(?i)my-company-api-key['\"]?\s*[:=]\s*['\"]?([a-zA-Z0-9]{32,})'''
secretGroup = 1

[allowlist]
paths = [
  '''\.github/workflows/.*\.yml$''',  # Don't scan workflow files
  '''test/.*''',
  '''.*_test\.go$''',
]

commits = [
  "abc123",  # Known false positive commits
]
```

## 🛡️ Pipeline Hardening

### Pinning Dependencies

```yaml
# Bad: Unpinned actions
- uses: actions/checkout@v4  # Could change!
- uses: actions/setup-node@latest  # Never use latest!

# Good: SHA-pinned actions
- uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11  # v4.1.1
- uses: actions/setup-node@60edb5dd545a775178f52524783378180af0d1f8  # v4.0.2
```

#### Renovate Configuration for Auto-Updates

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "config:base",
    "helpers:pinGitHubActionDigests"
  ],
  "github-actions": {
    "fileMatch": ["^\\.github/workflows/[^/]+\\.ya?ml$"],
    "pinDigests": true
  },
  "packageRules": [
    {
      "matchManagers": ["github-actions"],
      "matchUpdateTypes": ["minor", "patch"],
      "automerge": true
    }
  ]
}
```

### Runner Security

#### Self-Hosted Runner Hardening

```yaml
# Docker-based ephemeral runner
# docker-compose.yml
version: '3.8'
services:
  runner:
    image: myorg/github-runner:latest
    environment:
      - RUNNER_EPHEMERAL=true  # Destroy after each job
      - RUNNER_TOKEN=${RUNNER_TOKEN}
      - RUNNER_REPOSITORY=myorg/myrepo
    security_opt:
      - no-new-privileges:true
    read_only: true
    tmpfs:
      - /tmp:exec,size=2G
      - /home/runner:exec,size=10G
    cap_drop:
      - ALL
    networks:
      - runner-network

networks:
  runner-network:
    driver: bridge
    internal: true  # No internet access by default
```

#### Kubernetes-based Runners (Actions Runner Controller)

```yaml
# runner-deployment.yaml
apiVersion: actions.summerwind.dev/v1alpha1
kind: RunnerDeployment
metadata:
  name: secure-runner
spec:
  replicas: 3
  template:
    spec:
      repository: myorg/myrepo
      ephemeral: true
      dockerEnabled: false  # Disable Docker-in-Docker
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 1000
      resources:
        limits:
          cpu: "2"
          memory: "4Gi"
        requests:
          cpu: "500m"
          memory: "1Gi"
      tolerations:
        - key: "runner"
          operator: "Equal"
          value: "true"
          effect: "NoSchedule"
      nodeSelector:
        runner-pool: "secure"
```

### Workflow Permissions

```yaml
# Restrictive permissions (recommended)
permissions:
  contents: read
  packages: write
  id-token: write

# Or disable all and enable per-job
permissions: {}

jobs:
  build:
    permissions:
      contents: read
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
  
  deploy:
    permissions:
      id-token: write
      contents: read
    runs-on: ubuntu-latest
    needs: build
    steps:
      - uses: aws-actions/configure-aws-credentials@v4
```

### Branch Protection

```bash
# GitHub CLI to configure branch protection
gh api repos/{owner}/{repo}/branches/main/protection \
  -X PUT \
  -H "Accept: application/vnd.github+json" \
  -f required_status_checks='{"strict":true,"checks":[{"context":"build"},{"context":"test"},{"context":"security-scan"}]}' \
  -f enforce_admins=true \
  -f required_pull_request_reviews='{"dismiss_stale_reviews":true,"require_code_owner_reviews":true,"required_approving_review_count":2}' \
  -f restrictions=null \
  -f required_linear_history=true \
  -f allow_force_pushes=false \
  -f allow_deletions=false \
  -f required_signatures=true
```

## ✍️ Signed Commits & Attestations

### GPG Commit Signing

```bash
# Generate GPG key
gpg --full-generate-key

# List keys
gpg --list-secret-keys --keyid-format=long

# Export public key for GitHub
gpg --armor --export YOUR_KEY_ID

# Configure Git
git config --global user.signingkey YOUR_KEY_ID
git config --global commit.gpgsign true
git config --global tag.gpgsign true

# Sign a commit
git commit -S -m "Signed commit message"
```

### Sigstore/Gitsign (Keyless Signing)

```bash
# Install gitsign
brew install sigstore/tap/gitsign

# Configure Git to use gitsign
git config --global commit.gpgsign true
git config --global gpg.x509.program gitsign
git config --global gpg.format x509

# Commits are now signed using OIDC identity
git commit -m "Keyless signed commit"
```

### GitHub Artifact Attestations

```yaml
# .github/workflows/build-attest.yml
name: Build and Attest

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
      attestations: write
      packages: write

    steps:
      - uses: actions/checkout@v4

      - name: Build Docker image
        run: |
          docker build -t ghcr.io/${{ github.repository }}:${{ github.sha }} .

      - name: Login to GHCR
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Push image
        run: |
          docker push ghcr.io/${{ github.repository }}:${{ github.sha }}

      - name: Attest Build Provenance
        uses: actions/attest-build-provenance@v1
        with:
          subject-name: ghcr.io/${{ github.repository }}
          subject-digest: sha256:${{ steps.push.outputs.digest }}
          push-to-registry: true
```

## 🔍 Security Scanning Gates

### Comprehensive Security Pipeline

```yaml
# .github/workflows/security-pipeline.yml
name: Security Pipeline

on:
  pull_request:
  push:
    branches: [main]

jobs:
  # Static Application Security Testing
  sast:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run Semgrep
        uses: returntocorp/semgrep-action@v1
        with:
          config: >-
            p/security-audit
            p/secrets
            p/owasp-top-ten

  # Software Composition Analysis
  sca:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run Trivy SCA
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH'

      - name: Upload to GitHub Security
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: 'trivy-results.sarif'

  # Infrastructure as Code Scanning
  iac:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run Checkov
        uses: bridgecrewio/checkov-action@master
        with:
          directory: .
          framework: terraform,kubernetes,dockerfile
          soft_fail: false

  # Container Scanning
  container:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Build image
        run: docker build -t app:${{ github.sha }} .
      
      - name: Scan image
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'app:${{ github.sha }}'
          format: 'table'
          exit-code: '1'
          severity: 'CRITICAL,HIGH'
          ignore-unfixed: true

  # Security gate - all must pass
  security-gate:
    needs: [sast, sca, iac, container]
    runs-on: ubuntu-latest
    steps:
      - name: Security Gate Passed
        run: echo "All security checks passed!"
```

## 📝 GitLab CI Security

### GitLab Security Configuration

```yaml
# .gitlab-ci.yml
include:
  - template: Security/SAST.gitlab-ci.yml
  - template: Security/Secret-Detection.gitlab-ci.yml
  - template: Security/Dependency-Scanning.gitlab-ci.yml
  - template: Security/Container-Scanning.gitlab-ci.yml
  - template: Security/License-Scanning.gitlab-ci.yml

stages:
  - build
  - test
  - security
  - deploy

variables:
  # Secure variables
  SECURE_ANALYZERS_PREFIX: "registry.gitlab.com/security-products"
  SAST_EXCLUDED_PATHS: "spec, test, tests, tmp"
  SECRET_DETECTION_EXCLUDED_PATHS: ".git"

# Override to fail on HIGH+ vulnerabilities
sast:
  variables:
    SAST_EXCLUDED_ANALYZERS: ""
  rules:
    - if: $CI_MERGE_REQUEST_IID
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH

container_scanning:
  variables:
    CS_SEVERITY_THRESHOLD: HIGH

# Block deployment if vulnerabilities found
deploy_production:
  stage: deploy
  script:
    - ./deploy.sh
  environment:
    name: production
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
      when: manual
  dependencies:
    - container_scanning
    - dependency_scanning
```

## 🏭 Jenkins Security

### Jenkins Pipeline Security

```groovy
// Jenkinsfile
pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: builder
    image: docker:dind
    securityContext:
      privileged: false
      runAsNonRoot: true
      runAsUser: 1000
    resources:
      limits:
        memory: "2Gi"
        cpu: "1"
"""
        }
    }
    
    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
    }
    
    environment {
        // Use credentials binding
        AWS_CREDS = credentials('aws-production')
        DOCKER_REGISTRY = 'ghcr.io/myorg'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Security Scan') {
            parallel {
                stage('SAST') {
                    steps {
                        sh 'semgrep --config=auto .'
                    }
                }
                stage('Secret Scan') {
                    steps {
                        sh 'trufflehog filesystem . --only-verified'
                    }
                }
                stage('Dependency Scan') {
                    steps {
                        sh 'trivy fs . --exit-code 1 --severity HIGH,CRITICAL'
                    }
                }
            }
        }
        
        stage('Build') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-registry',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo $DOCKER_PASS | docker login ghcr.io -u $DOCKER_USER --password-stdin
                        docker build -t ${DOCKER_REGISTRY}/app:${BUILD_NUMBER} .
                        docker push ${DOCKER_REGISTRY}/app:${BUILD_NUMBER}
                    '''
                }
            }
        }
        
        stage('Container Scan') {
            steps {
                sh "trivy image ${DOCKER_REGISTRY}/app:${BUILD_NUMBER} --exit-code 1"
            }
        }
    }
    
    post {
        always {
            cleanWs()  // Clean workspace
        }
        failure {
            // Notify security team
            emailext (
                subject: "Security Pipeline Failed: ${env.JOB_NAME}",
                body: "Check ${env.BUILD_URL}",
                to: "security@company.com"
            )
        }
    }
}
```

## 📊 Audit & Monitoring

### Comprehensive Audit Logging

```yaml
# GitHub Actions audit logging
- name: Audit Log Entry
  run: |
    echo "::notice title=Deployment::Deploying to production by ${{ github.actor }}"
    
    # Send to SIEM
    curl -X POST https://siem.company.com/api/events \
      -H "Authorization: Bearer ${{ secrets.SIEM_TOKEN }}" \
      -d '{
        "event": "deployment",
        "actor": "${{ github.actor }}",
        "repository": "${{ github.repository }}",
        "commit": "${{ github.sha }}",
        "ref": "${{ github.ref }}",
        "workflow": "${{ github.workflow }}",
        "run_id": "${{ github.run_id }}",
        "timestamp": "'$(date -u +%Y-%m-%dT%H:%M:%SZ)'"
      }'
```

### Monitoring Dashboard Metrics

```yaml
# Prometheus metrics for CI/CD
# prometheus-ci-rules.yml
groups:
  - name: ci_cd_security
    rules:
      - alert: HighSecurityScanFailureRate
        expr: |
          rate(ci_security_scan_failures_total[1h]) / 
          rate(ci_security_scan_total[1h]) > 0.3
        for: 15m
        labels:
          severity: warning
        annotations:
          summary: "High security scan failure rate"
          
      - alert: SecretDetected
        expr: ci_secret_detection_findings > 0
        for: 0m
        labels:
          severity: critical
        annotations:
          summary: "Secret detected in codebase"
          
      - alert: CriticalVulnerabilityFound
        expr: ci_vulnerability_critical_count > 0
        for: 0m
        labels:
          severity: critical
        annotations:
          summary: "Critical vulnerability found in build"
```

## ✅ Security Checklist

### Pre-Deployment Checklist

- [ ] All secrets use external secret manager or OIDC
- [ ] Actions/dependencies are pinned to SHA
- [ ] Branch protection enabled with required reviews
- [ ] Required status checks configured
- [ ] Signed commits enforced
- [ ] Runners are ephemeral and isolated
- [ ] Network access is restricted
- [ ] Minimum required permissions set
- [ ] Security scanning gates configured
- [ ] Audit logging enabled
- [ ] Incident response plan documented

### Regular Review Checklist

- [ ] Review and rotate service account credentials (quarterly)
- [ ] Audit pipeline permissions (monthly)
- [ ] Review and update security scanning rules
- [ ] Test incident response procedures
- [ ] Update dependencies and security tools
- [ ] Review access logs for anomalies

## 🎯 Quick Reference

```bash
# Verify OIDC token (GitHub Actions)
curl -H "Authorization: Bearer $ACTIONS_ID_TOKEN_REQUEST_TOKEN" \
  "$ACTIONS_ID_TOKEN_REQUEST_URL&audience=sts.amazonaws.com"

# Check for hardcoded secrets
gitleaks detect --source . --verbose

# Verify image signature
cosign verify --key cosign.pub ghcr.io/org/image:tag

# Audit GitHub Actions usage
gh api repos/{owner}/{repo}/actions/runs --jq '.workflow_runs[] | {id, name, actor: .actor.login, created_at}'
```

---

**Next**: Learn about [Pipeline Scanning](./pipeline-scanning.md) for automated security testing.

