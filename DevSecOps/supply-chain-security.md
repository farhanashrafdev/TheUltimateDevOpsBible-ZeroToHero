# Supply Chain Security

## 🎯 Introduction

Software supply chain security protects against attacks targeting the components, processes, and tools used to build and deploy software.

## 📚 Attack Surface

```
┌─────────────────────────────────────────────────────────────────────┐
│                  Software Supply Chain Attack Vectors                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Source Code                                                      │
│     ├── Compromised developer accounts                              │
│     ├── Malicious commits/PRs                                       │
│     └── Insider threats                                             │
│                                                                      │
│  2. Dependencies                                                     │
│     ├── Typosquatting (malicious package names)                    │
│     ├── Dependency confusion                                        │
│     ├── Compromised maintainer accounts                            │
│     └── Abandoned packages with vulnerabilities                     │
│                                                                      │
│  3. Build Process                                                    │
│     ├── Compromised CI/CD pipelines (SolarWinds)                   │
│     ├── Malicious build scripts                                     │
│     └── Poisoned build caches                                       │
│                                                                      │
│  4. Artifacts                                                        │
│     ├── Unsigned/unverified images                                  │
│     ├── Compromised registries                                      │
│     └── Tampered artifacts in transit                               │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 🔐 SLSA Framework

SLSA (Supply chain Levels for Software Artifacts) provides a security framework with increasing levels of assurance:

```
Level 0: No guarantees
Level 1: Build process documented
Level 2: Hosted build platform, signed provenance
Level 3: Hardened build platform, non-falsifiable provenance
Level 4: Hermetic, reproducible builds
```

### SLSA Provenance with GitHub Actions

```yaml
name: SLSA Build

on:
  push:
    tags: ['v*']

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

      - name: Build and Push
        id: build
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ github.ref_name }}

      - name: Generate SLSA Provenance
        uses: actions/attest-build-provenance@v1
        with:
          subject-name: ghcr.io/${{ github.repository }}
          subject-digest: ${{ steps.build.outputs.digest }}
          push-to-registry: true
```

## 📦 Dependency Management

### Lock Files

```bash
# NPM - package-lock.json
npm ci  # Uses exact versions from lock file

# Python - requirements.txt with hashes
pip install --require-hashes -r requirements.txt

# Go - go.sum
go mod verify
```

### Pin Dependencies

```yaml
# requirements.txt with hashes
requests==2.31.0 \
    --hash=sha256:58cd2187c01e70e6e26505bca751777aa9f2ee0b7f4300988b709f44e013003f

# Dockerfile - pin base images by digest
FROM python:3.11@sha256:abc123...
```

### Private Registry

```yaml
# .npmrc for private registry
registry=https://npm.company.com/
//npm.company.com/:_authToken=${NPM_TOKEN}

# pip.conf
[global]
index-url = https://pypi.company.com/simple
trusted-host = pypi.company.com
```

## 🔍 SBOM (Software Bill of Materials)

### Generate SBOM

```yaml
- name: Generate SBOM
  uses: anchore/sbom-action@v0
  with:
    image: ghcr.io/${{ github.repository }}:${{ github.ref_name }}
    format: cyclonedx-json
    output-file: sbom.json
    
- name: Upload SBOM
  uses: actions/upload-artifact@v4
  with:
    name: sbom
    path: sbom.json
```

### Analyze SBOM

```bash
# Scan SBOM for vulnerabilities
grype sbom:sbom.json

# Check for licenses
syft packages sbom:sbom.json -o table
```

## 🛡️ Verification Pipeline

```yaml
name: Supply Chain Security

on:
  push:
    branches: [main]

jobs:
  verify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Verify commit signatures
      - name: Verify Commit Signature
        run: |
          git verify-commit HEAD || echo "Warning: Unsigned commit"

      # Scan dependencies
      - name: Dependency Scan
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          severity: 'CRITICAL,HIGH'

      # Check for dependency confusion
      - name: Check Package Names
        run: |
          # Ensure no internal packages reference public registries
          grep -r "registry.npmjs.org" package-lock.json && exit 1 || true

  build:
    needs: verify
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
      packages: write
      attestations: write

    steps:
      - uses: actions/checkout@v4
      
      - uses: docker/build-push-action@v5
        id: build
        with:
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ github.sha }}
          provenance: mode=max
          sbom: true

      - uses: sigstore/cosign-installer@v3
      
      - name: Sign Image
        run: cosign sign --yes ghcr.io/${{ github.repository }}@${{ steps.build.outputs.digest }}
```

## ✅ Best Practices

1. **Sign Everything**: Commits, images, SBOMs
2. **Pin Dependencies**: Use lock files and digests
3. **Generate SBOMs**: Track all components
4. **Verify Before Deploy**: Check signatures at admission
5. **Use Private Registries**: Control artifact sources
6. **Monitor Dependencies**: Continuous vulnerability scanning

---

**Next**: Learn about [Zero Trust](./zero-trust.md) architecture.

