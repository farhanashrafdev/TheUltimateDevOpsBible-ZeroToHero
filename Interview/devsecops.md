# DevSecOps Interview Questions: Complete Guide

## 🎯 Introduction

This comprehensive guide covers DevSecOps interview questions covering security integration, practices, and real-world scenarios.

## 🔒 Security Fundamentals

### What is DevSecOps?

**Answer**: DevSecOps integrates security into DevOps:

**Key Principles**:
1. **Shift-Left Security**: Security from the start
2. **Security as Code**: Infrastructure and policies as code
3. **Continuous Security**: Ongoing security practices
4. **Shared Responsibility**: Everyone owns security

**Benefits**:
- Early vulnerability detection
- Reduced security debt
- Faster remediation
- Compliance automation
- Security culture

### Explain SAST, SCA, DAST

**Answer**:

**SAST (Static Application Security Testing)**:
- Analyzes source code
- Finds vulnerabilities before code runs
- Examples: SonarQube, Checkmarx, Veracode
- Integrated in IDE and CI/CD

**SCA (Software Composition Analysis)**:
- Scans dependencies
- Finds known vulnerabilities
- License compliance
- Examples: Snyk, WhiteSource, Dependabot

**DAST (Dynamic Application Security Testing)**:
- Tests running applications
- Simulates real attacks
- Finds runtime vulnerabilities
- Examples: OWASP ZAP, Burp Suite

**When to use**:
- SAST: Early in development
- SCA: During build
- DAST: In staging/production

### How do you manage secrets?

**Answer**:

**Best Practices**:
1. **Secret Managers**: Vault, AWS Secrets Manager, Azure Key Vault
2. **Never Commit**: Use .gitignore, pre-commit hooks
3. **Rotate Regularly**: Automated rotation
4. **Least Privilege**: Minimal access
5. **Audit Access**: Log all secret access

**Example (Vault)**:
```bash
# Store secret
vault kv put secret/app api_key=abc123

# Retrieve in application
vault kv get secret/app
```

**Kubernetes**:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  api_key: base64encoded
```

## 🛡️ Security Practices

### How do you secure containers?

**Answer**:

**Build Time**:
- Secure Dockerfiles
- Minimal base images (Alpine, distroless)
- Multi-stage builds
- Non-root users

**Image Security**:
- Vulnerability scanning (Trivy, Snyk)
- Image signing (Cosign)
- SBOM generation

**Runtime Security**:
- Runtime protection (Falco)
- Network policies
- Pod security standards
- Resource limits

**Example**:
```dockerfile
FROM node:18-alpine
RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser
USER appuser
```

### Explain Supply Chain Security

**Answer**: Secure entire software supply chain:

**Areas**:
1. **Source Code**: Secure repos, access controls, code signing
2. **Dependencies**: Scanning, SBOM, license compliance
3. **Build Process**: Secure CI/CD, artifact signing, provenance
4. **Distribution**: Image signing, registry security, verification

**Tools**:
- **SBOM**: Syft, SPDX, CycloneDX
- **Signing**: Cosign, Notary
- **SLSA**: Supply chain levels

**Example**:
```bash
# Generate SBOM
syft packages docker:nginx:latest -o spdx-json > sbom.json

# Sign image
cosign sign user/image:tag
cosign verify user/image:tag
```

### What is Zero Trust?

**Answer**: Never trust, always verify:

**Principles**:
1. **Verify Explicitly**: Authenticate all requests
2. **Least Privilege**: Minimal access
3. **Assume Breach**: Segment and monitor

**Implementation**:
- Network segmentation
- Identity and access management
- Continuous monitoring
- Encryption everywhere

**Kubernetes Example**:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: zero-trust-policy
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: allowed-app
```

## 🔍 Security Scanning

### How do you implement security in CI/CD?

**Answer**:

**Security Gates**:
1. **Pre-commit**: Git hooks, secret scanning
2. **Build**: SAST, SCA, dependency scanning
3. **Test**: DAST, security testing
4. **Deploy**: Image scanning, IaC scanning, policy enforcement

**Example Pipeline**:
```yaml
stages:
  - build
  - test
  - security
  - deploy

security:
  stage: security
  script:
    - sonar-scanner  # SAST
    - snyk test      # SCA
    - trivy image    # Container scan
    - checkov -d .   # IaC scan
  allow_failure: false
```

### How do you scan Infrastructure as Code?

**Answer**:

**Tools**:
- **Checkov**: Terraform, CloudFormation, Kubernetes
- **Terrascan**: Multi-cloud support
- **tfsec**: Terraform focused

**Example**:
```bash
# Checkov
checkov -d terraform/

# Terrascan
terrascan scan -t terraform
```

**Common Checks**:
- Public S3 buckets
- Unencrypted storage
- Open security groups
- Missing IAM policies
- Hardcoded secrets

## 🎯 Scenario Questions

### Design a secure container deployment

**Answer**:

1. **Image Security**:
   - Scan images (Trivy)
   - Sign images (Cosign)
   - Minimal base images
   - Non-root users

2. **Runtime Security**:
   - Runtime protection (Falco)
   - Network policies
   - Pod security standards
   - RBAC

3. **Secrets Management**:
   - External secret managers
   - Kubernetes secrets (encrypted)
   - Rotation policies

4. **Monitoring**:
   - Security events
   - Anomaly detection
   - Compliance monitoring

**Example**:
```yaml
apiVersion: v1
kind: Pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
  containers:
  - name: app
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
          - ALL
```

### How do you handle a security incident?

**Answer**:

1. **Detection**: Identify security event
2. **Containment**: Isolate affected systems
3. **Investigation**: Determine scope and impact
4. **Remediation**: Fix vulnerabilities
5. **Recovery**: Restore services
6. **Post-Mortem**: Document and learn
7. **Prevention**: Implement measures

**Best Practices**:
- Incident response plan
- Security team coordination
- Communication plan
- Forensic analysis
- Blameless culture

## ✅ Key Areas to Master

- [ ] Security scanning (SAST/SCA/DAST)
- [ ] Secrets management
- [ ] Container security
- [ ] Supply chain security
- [ ] IaC security
- [ ] CI/CD security
- [ ] Zero trust
- [ ] Compliance
- [ ] Incident response
- [ ] Security best practices

---

**Remember**: DevSecOps interviews test security knowledge and integration practices. Be prepared to discuss security tools, practices, and real-world scenarios. Show understanding of security throughout the SDLC.
