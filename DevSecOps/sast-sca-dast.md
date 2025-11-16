# SAST, SCA, DAST: Complete Security Testing Guide

## 🎯 Introduction

Security testing is crucial for identifying vulnerabilities before they reach production. This guide covers the three main types of security testing: SAST, SCA, and DAST.

### Security Testing Types

```
┌─────────────────────────────────────────┐
│         Security Testing Pyramid          │
│                                           │
│              ┌─────────┐                 │
│              │  DAST   │ ← Runtime        │
│              └─────────┘                   │
│            ┌─────────────┐                │
│            │     SCA      │ ← Dependencies│
│            └─────────────┘                │
│          ┌─────────────────┐              │
│          │      SAST       │ ← Source Code│
│          └─────────────────┘              │
└─────────────────────────────────────────┘
```

## 🔍 SAST (Static Application Security Testing)

### What is SAST?

**SAST** analyzes source code, bytecode, or binary code to find security vulnerabilities without executing the application.

**Key Characteristics**:
- Analyzes code at rest
- No running application needed
- Finds issues early in SDLC
- Can analyze entire codebase
- Language-specific tools

### How SAST Works

```
Source Code → SAST Tool → Vulnerability Report
     │
     ├── Pattern Matching
     ├── Data Flow Analysis
     ├── Control Flow Analysis
     └── Taint Analysis
```

### SAST Tools

#### SonarQube

**Popular open-source SAST tool**

```yaml
# GitHub Actions Example
name: SonarQube Scan
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  sonarqube:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0
      
      - name: SonarQube Scan
        uses: sonarsource/sonarqube-scan-action@master
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
```

**SonarQube Configuration**:

```properties
# sonar-project.properties
sonar.projectKey=my-project
sonar.sources=src
sonar.tests=test
sonar.language=java
sonar.sourceEncoding=UTF-8
sonar.exclusions=**/node_modules/**,**/dist/**
```

#### Checkmarx

**Enterprise SAST solution**

```bash
# Checkmarx CLI
checkmarx scan \
  --project-name "my-project" \
  --source-dir ./src \
  --output-format json \
  --output-file results.json
```

#### Veracode

**Cloud-based SAST**

```yaml
# Veracode Pipeline Scan
- name: Veracode SAST Scan
  uses: veracode/veracode-pipeline-scan@main
  with:
    vid: ${{ secrets.VERACODE_API_ID }}
    vkey: ${{ secrets.VERACODE_API_KEY }}
```

### SAST Best Practices

#### 1. Integrate Early

```yaml
# Pre-commit hooks
# IDE plugins
# CI/CD integration
```

#### 2. Configure Properly

```yaml
# Exclude false positives
# Set severity thresholds
# Custom rules
# Regular updates
```

#### 3. Fix Critical Issues

```yaml
# Prioritize by severity
# Fix critical first
# Track remediation
# Regular reviews
```

## 📦 SCA (Software Composition Analysis)

### What is SCA?

**SCA** scans dependencies and third-party libraries for known vulnerabilities, license compliance issues, and outdated packages.

**Key Characteristics**:
- Scans dependencies
- Checks vulnerability databases
- License compliance
- Update recommendations
- SBOM generation

### How SCA Works

```
Dependencies → SCA Tool → Vulnerability DB → Report
     │
     ├── Package.json / requirements.txt
     ├── Lock files
     ├── Container images
     └── Binary analysis
```

### SCA Tools

#### Snyk

**Popular SCA tool**

```bash
# Install Snyk
npm install -g snyk

# Authenticate
snyk auth $SNYK_TOKEN

# Test dependencies
snyk test

# Monitor project
snyk monitor

# Fix vulnerabilities
snyk test --json | snyk-to-html -o results.html
```

**GitHub Actions Integration**:

```yaml
name: Snyk Security Scan
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Run Snyk to check for vulnerabilities
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high
      
      - name: Upload Snyk results to GitHub Security
        uses: github/codeql-action/upload-sarif@v2
        if: always()
        with:
          sarif_file: snyk.sarif
```

#### Dependabot

**GitHub-native dependency management**

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
    reviewers:
      - "team-security"
    labels:
      - "dependencies"
      - "security"
```

#### WhiteSource (Mend)

**Enterprise SCA solution**

```yaml
# WhiteSource Action
- name: WhiteSource Security Scan
  uses: whitesource/run-action@master
  with:
    user_key: ${{ secrets.WS_USER_KEY }}
    api_key: ${{ secrets.WS_API_KEY }}
    product_name: "my-product"
```

### SCA Best Practices

#### 1. Regular Scanning

```yaml
# Daily scans
# Weekly dependency updates
# Automated PRs
# Security alerts
```

#### 2. License Compliance

```yaml
# Check licenses
# Policy enforcement
# Approved licenses list
# License tracking
```

#### 3. Update Dependencies

```yaml
# Regular updates
# Security patches
# Automated updates
# Testing updates
```

## 🎯 DAST (Dynamic Application Security Testing)

### What is DAST?

**DAST** tests running applications by simulating attacks and finding runtime vulnerabilities.

**Key Characteristics**:
- Tests running applications
- Simulates real attacks
- Finds runtime issues
- No source code needed
- Black-box testing

### How DAST Works

```
Running App → DAST Tool → Attack Simulation → Vulnerability Report
     │
     ├── HTTP Requests
     ├── Parameter Manipulation
     ├── Authentication Testing
     └── Session Management
```

### DAST Tools

#### OWASP ZAP

**Open-source DAST tool**

```bash
# Install ZAP
docker pull owasp/zap2docker-stable

# Baseline scan
docker run -t owasp/zap2docker-stable zap-baseline.py \
  -t https://example.com

# Full scan
docker run -t owasp/zap2docker-stable zap-full-scan.py \
  -t https://example.com \
  -J zap-report.json
```

**GitHub Actions Integration**:

```yaml
name: OWASP ZAP Scan
on:
  push:
    branches: [ main ]

jobs:
  zap_scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: ZAP Baseline Scan
        uses: zaproxy/action-baseline@v0.7.0
        with:
          target: 'https://example.com'
          rules_file_name: '.zap/rules.tsv'
          cmd_options: '-a'
      
      - name: Upload ZAP results
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: zap-results
          path: zap-results/
```

#### Burp Suite

**Professional DAST tool**

```bash
# Burp Suite Professional
# GUI-based testing
# Advanced scanning
# Manual testing support
```

#### Nuclei

**Fast vulnerability scanner**

```bash
# Install Nuclei
go install -v github.com/projectdiscovery/nuclei/v2/cmd/nuclei@latest

# Scan target
nuclei -u https://example.com

# Use templates
nuclei -u https://example.com -t nuclei-templates/

# Output JSON
nuclei -u https://example.com -o results.json -json
```

### DAST Best Practices

#### 1. Test in Staging

```yaml
# Test before production
# Staging environment
# Production-like data
# Safe testing
```

#### 2. Regular Scans

```yaml
# Scheduled scans
# After deployments
# Weekly scans
# Continuous monitoring
```

#### 3. Comprehensive Testing

```yaml
# All endpoints
# Authentication flows
# API endpoints
# Web applications
```

## 🔄 Integrating All Three

### Complete Security Pipeline

```yaml
name: Complete Security Scan
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  sast:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: SonarQube Scan
        uses: sonarsource/sonarqube-scan-action@master
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
  
  sca:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Snyk Scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
  
  dast:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: ZAP Scan
        uses: zaproxy/action-baseline@v0.7.0
        with:
          target: 'https://staging.example.com'
  
  security-gate:
    needs: [sast, sca, dast]
    runs-on: ubuntu-latest
    if: failure()
    steps:
      - name: Block deployment
        run: exit 1
```

## 📊 Security Testing Metrics

### Key Metrics

- **Vulnerability Density**: Vulnerabilities per 1000 LOC
- **Time to Fix**: Average time to fix vulnerabilities
- **False Positive Rate**: Percentage of false positives
- **Coverage**: Percentage of code tested
- **Remediation Rate**: Percentage of issues fixed

### Reporting

```yaml
# Security Dashboard
- Open vulnerabilities by severity
- Vulnerability trends
- Remediation progress
- Compliance status
```

## ✅ Best Practices

### 1. Use All Three Types

```yaml
# SAST for code analysis
# SCA for dependencies
# DAST for runtime testing
# Complementary coverage
```

### 2. Integrate in CI/CD

```yaml
# Automated scanning
# Security gates
# Fail on critical issues
# Fast feedback
```

### 3. Regular Scans

```yaml
# Daily scans
# Weekly comprehensive scans
# After every change
# Continuous monitoring
```

### 4. Fix Critical Issues

```yaml
# Prioritize by severity
# Fix critical first
# Track remediation
# Regular reviews
```

### 5. Track Remediation

```yaml
# Vulnerability tracking
# Remediation timelines
# Progress reporting
# SLA compliance
```

## ✅ Mastery Checklist

- [ ] Understand SAST, SCA, DAST
- [ ] Set up SAST scanning
- [ ] Configure SCA tools
- [ ] Implement DAST testing
- [ ] Integrate in CI/CD
- [ ] Fix vulnerabilities
- [ ] Track remediation
- [ ] Monitor metrics
- [ ] Regular reviews
- [ ] Continuous improvement

---

**Next Steps**:
- Learn [CI/CD Security](./ci-cd-security.md)
- Explore [Container Security](./container-security.md)
- Master [Secrets Management](./secrets-management.md)

**Remember**: Security testing is not optional. Use SAST for code analysis, SCA for dependencies, and DAST for runtime testing. Integrate all three in your CI/CD pipeline and fix critical issues before deployment.
