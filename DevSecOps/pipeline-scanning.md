# Pipeline Scanning

## 🎯 Introduction

Pipeline scanning integrates automated security testing directly into your CI/CD workflow, providing continuous security validation at every stage of development. This shift-left approach catches vulnerabilities early when they're cheapest and easiest to fix.

### Security Scanning Pyramid

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Pipeline Security Scanning                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│                         Production                                   │
│                            ▲                                         │
│                           ╱│╲                                        │
│                          ╱ │ ╲   DAST (Dynamic)                     │
│                         ╱  │  ╲  Runtime Security                   │
│                        ╱───┼───╲                                    │
│                       ╱    │    ╲                                   │
│                      ╱  Container ╲   Image Scanning                │
│                     ╱   Scanning   ╲  SBOM Generation               │
│                    ╱───────┼────────╲                               │
│                   ╱        │         ╲                              │
│                  ╱      IaC Scan      ╲  Terraform, K8s             │
│                 ╱    Policy as Code    ╲ CloudFormation             │
│                ╱───────────┼────────────╲                           │
│               ╱            │             ╲                          │
│              ╱          SCA Scan          ╲  Dependencies           │
│             ╱     License Compliance       ╲ SBOM                   │
│            ╱───────────────┼────────────────╲                       │
│           ╱                │                 ╲                      │
│          ╱             SAST Scan              ╲  Source Code        │
│         ╱        Secret Detection              ╲ Patterns           │
│        ╱─────────────────────────────────────────╲                  │
│        │                                          │                  │
│        │              Pre-commit                  │  Local           │
│        │          Hooks & IDE Plugins             │  Development     │
│        └──────────────────────────────────────────┘                  │
│                                                                      │
│   Earlier Detection = Lower Cost to Fix                              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 📚 Scanning Types

### 1. SAST (Static Application Security Testing)

SAST analyzes source code without executing it to find vulnerabilities.

#### Semgrep Configuration

```yaml
# .semgrep.yml
rules:
  - id: hardcoded-password
    patterns:
      - pattern-either:
          - pattern: password = "..."
          - pattern: PASSWORD = "..."
          - pattern: passwd = "..."
    message: "Hardcoded password detected"
    severity: ERROR
    languages: [python, javascript, java]

  - id: sql-injection
    patterns:
      - pattern: |
          $QUERY = "..." + $USER_INPUT + "..."
          $DB.execute($QUERY)
    message: "Potential SQL injection vulnerability"
    severity: ERROR
    languages: [python]
    
  - id: insecure-deserialization
    pattern: pickle.loads($DATA)
    message: "Insecure deserialization with pickle"
    severity: WARNING
    languages: [python]
```

#### GitHub Actions SAST Pipeline

```yaml
# .github/workflows/sast.yml
name: SAST Scanning

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  semgrep:
    runs-on: ubuntu-latest
    container:
      image: returntocorp/semgrep
    steps:
      - uses: actions/checkout@v4
      
      - name: Run Semgrep
        run: |
          semgrep ci \
            --config=auto \
            --config=p/security-audit \
            --config=p/secrets \
            --config=p/owasp-top-ten \
            --sarif --output=semgrep.sarif
            
      - name: Upload SARIF
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: semgrep.sarif
          
  codeql:
    runs-on: ubuntu-latest
    permissions:
      security-events: write
    steps:
      - uses: actions/checkout@v4
      
      - name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: javascript, python
          
      - name: Autobuild
        uses: github/codeql-action/autobuild@v3
        
      - name: Perform Analysis
        uses: github/codeql-action/analyze@v3
```

#### SonarQube Integration

```yaml
# .github/workflows/sonarqube.yml
name: SonarQube Analysis

on:
  push:
    branches: [main]
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  sonarqube:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for blame
          
      - name: SonarQube Scan
        uses: sonarsource/sonarqube-scan-action@master
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
        with:
          args: >
            -Dsonar.projectKey=my-project
            -Dsonar.sources=src
            -Dsonar.tests=tests
            -Dsonar.python.coverage.reportPaths=coverage.xml
            
      - name: Quality Gate
        uses: sonarsource/sonarqube-quality-gate-action@master
        timeout-minutes: 5
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

**sonar-project.properties:**

```properties
sonar.projectKey=my-project
sonar.projectName=My Project
sonar.projectVersion=1.0

sonar.sources=src
sonar.tests=tests
sonar.exclusions=**/node_modules/**,**/vendor/**

sonar.python.coverage.reportPaths=coverage.xml
sonar.python.xunit.reportPath=test-results.xml

# Security hotspots
sonar.security.hotspots.inheritedRules=true

# Quality gates
sonar.qualitygate.wait=true
```

### 2. SCA (Software Composition Analysis)

SCA scans dependencies for known vulnerabilities and license issues.

#### Trivy SCA Scanning

```yaml
# .github/workflows/sca.yml
name: Dependency Scanning

on:
  push:
    branches: [main]
  pull_request:
  schedule:
    - cron: '0 0 * * *'  # Daily scan

jobs:
  trivy-sca:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH'
          ignore-unfixed: true
          
      - name: Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: 'trivy-results.sarif'
          
  snyk:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run Snyk
        uses: snyk/actions/node@master
        continue-on-error: true
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high --sarif-file-output=snyk.sarif
          
      - name: Upload Snyk results
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: snyk.sarif
```

#### Dependency Track SBOM

```yaml
# Generate and upload SBOM
- name: Generate SBOM
  uses: anchore/sbom-action@v0
  with:
    path: ./
    format: cyclonedx-json
    output-file: sbom.json
    
- name: Upload to Dependency Track
  run: |
    curl -X POST \
      -H "X-API-Key: ${{ secrets.DTRACK_API_KEY }}" \
      -H "Content-Type: application/json" \
      -d @sbom.json \
      "${{ secrets.DTRACK_URL }}/api/v1/bom"
```

### 3. Secret Detection

#### Pre-commit Secret Scanning

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.0
    hooks:
      - id: gitleaks
        
  - repo: https://github.com/trufflesecurity/trufflehog
    rev: v3.63.0
    hooks:
      - id: trufflehog
        entry: trufflehog git file://. --only-verified --fail
        
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.4.0
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']
```

#### Pipeline Secret Scanning

```yaml
# .github/workflows/secrets.yml
name: Secret Detection

on:
  push:
  pull_request:

jobs:
  gitleaks:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
          
      - name: Gitleaks
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          GITLEAKS_NOTIFY_USER_LIST: '@security-team'
          
  trufflehog:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
          
      - name: TruffleHog
        uses: trufflesecurity/trufflehog@main
        with:
          path: ./
          base: ${{ github.event.repository.default_branch }}
          head: HEAD
          extra_args: --only-verified
```

### 4. Container Scanning

#### Multi-Stage Container Scanning

```yaml
# .github/workflows/container-scan.yml
name: Container Security

on:
  push:
    paths:
      - 'Dockerfile*'
      - '.github/workflows/container-scan.yml'

jobs:
  dockerfile-lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Hadolint
        uses: hadolint/hadolint-action@v3.1.0
        with:
          dockerfile: Dockerfile
          failure-threshold: error
          
  build-scan:
    needs: dockerfile-lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Build image
        run: |
          docker build -t app:${{ github.sha }} .
          
      - name: Trivy Image Scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'app:${{ github.sha }}'
          format: 'sarif'
          output: 'trivy-image.sarif'
          severity: 'CRITICAL,HIGH'
          
      - name: Grype Scan
        uses: anchore/scan-action@v3
        with:
          image: 'app:${{ github.sha }}'
          fail-build: true
          severity-cutoff: high
          
      - name: Upload results
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: 'trivy-image.sarif'
```

### 5. IaC Scanning

```yaml
# .github/workflows/iac-scan.yml
name: Infrastructure Security

on:
  push:
    paths:
      - '**/*.tf'
      - '**/*.yaml'
      - '**/*.yml'
      - 'kubernetes/**'

jobs:
  terraform-security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: tfsec
        uses: aquasecurity/tfsec-action@v1.0.0
        with:
          soft_fail: false
          
      - name: Checkov
        uses: bridgecrewio/checkov-action@master
        with:
          directory: .
          framework: terraform
          output_format: sarif
          output_file_path: checkov.sarif
          
      - name: Terrascan
        uses: tenable/terrascan-action@main
        with:
          iac_type: 'terraform'
          iac_version: 'v14'
          policy_type: 'aws'
          sarif_upload: true
          
  kubernetes-security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Kubesec
        uses: controlplaneio/kubesec-action@master
        with:
          input: kubernetes/
          
      - name: Kube-linter
        uses: stackrox/kube-linter-action@v1
        with:
          directory: kubernetes/
          config: .kube-linter.yaml
```

## 🔧 Unified Security Pipeline

### Complete Pipeline Example

```yaml
# .github/workflows/security-pipeline.yml
name: Complete Security Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
  schedule:
    - cron: '0 2 * * *'  # Nightly

env:
  SCAN_RESULTS_DIR: security-results

jobs:
  # Stage 1: Quick checks
  pre-flight:
    runs-on: ubuntu-latest
    outputs:
      has-code-changes: ${{ steps.changes.outputs.code }}
      has-infra-changes: ${{ steps.changes.outputs.infra }}
    steps:
      - uses: actions/checkout@v4
      
      - uses: dorny/paths-filter@v2
        id: changes
        with:
          filters: |
            code:
              - 'src/**'
              - '*.py'
              - '*.js'
            infra:
              - '**/*.tf'
              - 'kubernetes/**'
              
  # Stage 2: Static Analysis
  sast:
    needs: pre-flight
    if: needs.pre-flight.outputs.has-code-changes == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Semgrep
        uses: returntocorp/semgrep-action@v1
        with:
          config: >-
            p/security-audit
            p/secrets
            p/owasp-top-ten
          generateSarif: true
          
      - name: Upload SARIF
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: semgrep.sarif

  # Stage 3: Dependency Analysis
  sca:
    needs: pre-flight
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Trivy FS Scan
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          format: 'sarif'
          output: 'trivy-sca.sarif'
          
      - name: Generate SBOM
        uses: anchore/sbom-action@v0
        with:
          format: spdx-json
          output-file: sbom.spdx.json
          
      - name: Upload SBOM
        uses: actions/upload-artifact@v4
        with:
          name: sbom
          path: sbom.spdx.json

  # Stage 4: Secret Detection
  secrets:
    needs: pre-flight
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
          
      - name: Gitleaks
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

  # Stage 5: IaC Security
  iac:
    needs: pre-flight
    if: needs.pre-flight.outputs.has-infra-changes == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Checkov
        uses: bridgecrewio/checkov-action@master
        with:
          directory: .
          framework: terraform,kubernetes
          output_format: sarif
          output_file_path: checkov.sarif

  # Stage 6: Container Security
  container:
    needs: [sast, sca, secrets]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Build image
        run: docker build -t app:${{ github.sha }} .
        
      - name: Trivy Image Scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'app:${{ github.sha }}'
          format: 'sarif'
          output: 'trivy-container.sarif'
          severity: 'CRITICAL,HIGH'
          ignore-unfixed: true

  # Stage 7: Security Gate
  security-gate:
    needs: [sast, sca, secrets, iac, container]
    if: always()
    runs-on: ubuntu-latest
    steps:
      - name: Check results
        run: |
          if [ "${{ needs.sast.result }}" == "failure" ] || \
             [ "${{ needs.secrets.result }}" == "failure" ]; then
            echo "Critical security checks failed!"
            exit 1
          fi
          echo "All security checks passed!"
          
      - name: Post summary
        run: |
          echo "## Security Scan Summary" >> $GITHUB_STEP_SUMMARY
          echo "| Check | Status |" >> $GITHUB_STEP_SUMMARY
          echo "|-------|--------|" >> $GITHUB_STEP_SUMMARY
          echo "| SAST | ${{ needs.sast.result }} |" >> $GITHUB_STEP_SUMMARY
          echo "| SCA | ${{ needs.sca.result }} |" >> $GITHUB_STEP_SUMMARY
          echo "| Secrets | ${{ needs.secrets.result }} |" >> $GITHUB_STEP_SUMMARY
          echo "| IaC | ${{ needs.iac.result }} |" >> $GITHUB_STEP_SUMMARY
          echo "| Container | ${{ needs.container.result }} |" >> $GITHUB_STEP_SUMMARY
```

## 📊 Reporting & Metrics

### Security Dashboard

```yaml
# Generate security metrics
- name: Generate Metrics
  run: |
    echo "security_scans_total $(date +%s)" >> metrics.txt
    echo "security_findings_critical $(cat results/*.sarif | jq '[.runs[].results[] | select(.level == "error")] | length')" >> metrics.txt
    echo "security_findings_high $(cat results/*.sarif | jq '[.runs[].results[] | select(.level == "warning")] | length')" >> metrics.txt

- name: Push to Prometheus
  run: |
    cat metrics.txt | while read metric value; do
      curl -X POST "http://pushgateway:9091/metrics/job/security-pipeline" \
        --data-binary "${metric} ${value}"
    done
```

### Notification Integration

```yaml
- name: Notify on Findings
  if: failure()
  uses: slackapi/slack-github-action@v1
  with:
    payload: |
      {
        "text": "🚨 Security vulnerabilities found in ${{ github.repository }}",
        "blocks": [
          {
            "type": "section",
            "text": {
              "type": "mrkdwn",
              "text": "*Security Alert*\nVulnerabilities detected in <${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}|workflow run>"
            }
          }
        ]
      }
  env:
    SLACK_WEBHOOK_URL: ${{ secrets.SLACK_SECURITY_WEBHOOK }}
```

## ✅ Best Practices

### Scanning Strategy

1. **Shift Left**: Run lightweight scans in pre-commit hooks
2. **Fail Fast**: Critical vulnerabilities should block builds
3. **Prioritize**: Not all findings are equal - focus on exploitable issues
4. **Automate**: Security shouldn't require manual intervention
5. **Track Trends**: Monitor vulnerability counts over time

### Tool Selection Matrix

| Need | Recommended Tool | Alternative |
|------|-----------------|-------------|
| SAST | Semgrep | CodeQL, SonarQube |
| SCA | Trivy | Snyk, Dependabot |
| Secrets | Gitleaks | TruffleHog |
| Containers | Trivy | Grype, Clair |
| IaC | Checkov | tfsec, Terrascan |
| DAST | OWASP ZAP | Burp Suite |

---

**Next**: Learn about [SAST/SCA/DAST](./sast-sca-dast.md) scanning in detail.

