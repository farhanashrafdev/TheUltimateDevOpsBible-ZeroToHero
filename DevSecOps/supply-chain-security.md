# Supply Chain Security: Protecting the Software Supply Chain

## 🎯 Introduction

Modern applications depend on hundreds of third-party libraries. A compromised dependency = compromised application. Supply chain attacks are increasing (SolarWinds, Log4Shell, etc.).

## 🔐 SLSA Framework

**Supply-chain Levels for Software Artifacts (SLSA)** - pronounced "salsa"

### SLSA Levels

| Level | Requirements | Example |
|:---|:---|:---|
| **SLSA 1** | Build process documented | README with build instructions |
| **SLSA 2** | Version control + build service | GitHub Actions, GitLab CI |
| **SLSA 3** | Hardened build, provenance | Signed provenance, isolated builds |
| **SLSA 4** | Hermetic builds, two-party review | Reproducible builds, code review |

### Implementing SLSA 3

```yaml
# GitHub Actions with SLSA provenance
name: SLSA Build

on: [push]

permissions:
  contents: read
  id-token: write
  packages: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build
        run: npm run build
        
      - name: Generate provenance
        uses: slsa-framework/slsa-github-generator@v1
        with:
          artifact-path: dist/
          
      - name: Verify provenance
        run: slsa-verifier verify-artifact dist/app.tar.gz --provenance-path provenance.json
```

---

## 📦 SBOM (Software Bill of Materials)

A list of all components in your software.

### Generate SBOM

```bash
# Using Syft
syft myapp:latest -o spdx-json > sbom.json
syft myapp:latest -o cyclonedx-json > sbom.json

# Using Trivy
trivy image --format cyclonedx myapp:latest > sbom.json

# Using Docker
docker sbom myapp:latest
```

### SBOM Formats

- **SPDX**: Linux Foundation standard
- **CycloneDX**: OWASP standard
- Both are widely supported

### Verify SBOM

```bash
# Attest SBOM with Cosign
cosign attest --predicate sbom.json --key cosign.key myapp:latest

# Verify SBOM attestation
cosign verify-attestation --key cosign.pub myapp:latest
```

---

## 🔍 Dependency Scanning

### Scan for Vulnerabilities

```bash
# NPM
npm audit
npm audit fix

# Python
pip-audit
safety check

# Go
govulncheck ./...

# Multi-language
snyk test
trivy fs .
```

### Automated Dependency Updates

**Dependabot (GitHub)**:
```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
```

**Renovate**:
```json
{
  "extends": ["config:base"],
  "automerge": true,
  "automergeType": "pr",
  "automergeStrategy": "squash"
}
```

---

## 🔒 Artifact Signing

### Sign All Artifacts

```bash
# Sign container image
cosign sign myapp:v1.0.0

# Sign binary
cosign sign-blob --key cosign.key binary.exe > binary.sig

# Sign Helm chart
helm package mychart/
cosign sign mychart-1.0.0.tgz
```

---

## 🛡️ Sigstore Ecosystem

### Components

1. **Cosign**: Sign and verify artifacts
2. **Rekor**: Transparency log
3. **Fulcio**: Certificate authority (keyless signing)

### Keyless Signing

```bash
# Sign without managing keys
cosign sign myapp:v1.0.0

# Verify with certificate identity
cosign verify \
  --certificate-identity=user@example.com \
  --certificate-oidc-issuer=https://github.com/login/oauth \
  myapp:v1.0.0
```

---

## 🎯 Complete Supply Chain Security

```yaml
name: Secure Supply Chain

on: [push]

permissions:
  contents: read
  id-token: write
  packages: write

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      # 1. Dependency scanning
      - name: Scan dependencies
        run: |
          npm audit --audit-level=high
          snyk test --severity-threshold=high
          
      # 2. Build
      - name: Build
        run: npm run build
        
      # 3. Generate SBOM
      - name: Generate SBOM
        run: syft . -o spdx-json > sbom.json
        
      # 4. Build container
      - name: Build image
        run: docker build -t myapp:${{ github.sha }} .
        
      # 5. Scan container
      - name: Scan image
        run: trivy image --severity HIGH,CRITICAL myapp:${{ github.sha }}
        
      # 6. Sign image
      - name: Sign image
        run: cosign sign myapp:${{ github.sha }}
        
      # 7. Attest SBOM
      - name: Attest SBOM
        run: cosign attest --predicate sbom.json myapp:${{ github.sha }}
        
      # 8. Generate provenance
      - name: Generate provenance
        uses: slsa-framework/slsa-github-generator@v1
        
      # 9. Push
      - name: Push image
        run: docker push myapp:${{ github.sha }}
```

---

## ✅ Best Practices

- [ ] Generate SBOM for all artifacts
- [ ] Sign all artifacts (containers, binaries, charts)
- [ ] Scan dependencies regularly
- [ ] Automate dependency updates
- [ ] Implement SLSA Level 3+
- [ ] Use transparency logs (Rekor)
- [ ] Verify signatures before deployment
- [ ] Monitor for vulnerabilities
- [ ] Document your supply chain

---

## 📚 Tools

- **SBOM**: Syft, Trivy, CycloneDX
- **Signing**: Cosign, Sigstore
- **Scanning**: Snyk, Trivy, Grype
- **Provenance**: SLSA generators
- **Updates**: Dependabot, Renovate

---

**Next**: Learn about [Zero Trust](./zero-trust.md) and [K8s Runtime Security](./k8s-runtime-security.md).
