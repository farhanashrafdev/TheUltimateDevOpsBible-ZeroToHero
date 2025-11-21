# Image Signing: Ensuring Container Integrity

## 🎯 Introduction

How do you know the container image you're deploying is the one you built? Image signing provides cryptographic proof of authenticity and integrity.

## 🔐 Why Sign Images?

**Without Signing**:
- Anyone can push malicious images to your registry
- No way to verify image hasn't been tampered with
- Supply chain attacks are easy

**With Signing**:
- Cryptographic proof of who built the image
- Detect tampering
- Enforce policy (only deploy signed images)

---

## 🛠️ Cosign: The Standard

Cosign is part of the Sigstore project and is the industry standard for container signing.

### Installation

```bash
# macOS
brew install cosign

# Linux
wget https://github.com/sigstore/cosign/releases/latest/download/cosign-linux-amd64
chmod +x cosign-linux-amd64
sudo mv cosign-linux-amd64 /usr/local/bin/cosign
```

### Generate Keys

```bash
# Generate key pair
cosign generate-key-pair

# Output:
# - cosign.key (private key - keep secret!)
# - cosign.pub (public key - share this)
```

### Sign an Image

```bash
# Build image
docker build -t myapp:v1.0.0 .
docker push myapp:v1.0.0

# Sign image
cosign sign --key cosign.key myapp:v1.0.0

# Verify signature
cosign verify --key cosign.pub myapp:v1.0.0
```

---

## 🔑 Keyless Signing (OIDC)

No need to manage keys! Use your identity provider (GitHub, Google, etc.).

```bash
# Sign with keyless (uses OIDC)
cosign sign myapp:v1.0.0

# This will:
# 1. Open browser for authentication
# 2. Sign with ephemeral key
# 3. Store signature in Rekor (transparency log)

# Verify keyless signature
cosign verify \
  --certificate-identity=user@example.com \
  --certificate-oidc-issuer=https://github.com/login/oauth \
  myapp:v1.0.0
```

---

## 🏭 CI/CD Integration

### GitHub Actions

```yaml
name: Build and Sign

on: [push]

permissions:
  contents: read
  packages: write
  id-token: write  # Required for keyless signing

jobs:
  build-and-sign:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Install Cosign
        uses: sigstore/cosign-installer@v3
        
      - name: Build image
        run: |
          docker build -t ghcr.io/${{ github.repository }}:${{ github.sha }} .
          docker push ghcr.io/${{ github.repository }}:${{ github.sha }}
          
      - name: Sign image (keyless)
        run: |
          cosign sign ghcr.io/${{ github.repository }}:${{ github.sha }}
```

### GitLab CI

```yaml
sign-image:
  stage: sign
  image: gcr.io/projectsigstore/cosign:latest
  script:
    - cosign sign --key $COSIGN_KEY $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
```

---

## 🔒 Policy Enforcement

### Kubernetes Admission Controller

Use **Kyverno** or **OPA Gatekeeper** to enforce signed images.

**Kyverno Policy**:
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: verify-image-signature
spec:
  validationFailureAction: enforce
  rules:
    - name: verify-signature
      match:
        any:
          - resources:
              kinds:
                - Pod
      verifyImages:
        - imageReferences:
            - "ghcr.io/myorg/*"
          attestors:
            - count: 1
              entries:
                - keys:
                    publicKeys: |-
                      -----BEGIN PUBLIC KEY-----
                      ...
                      -----END PUBLIC KEY-----
```

**Result**: Pods with unsigned images are rejected.

---

## 📦 Attestations

Sign not just the image, but also metadata (SBOM, provenance, test results).

### Sign SBOM

```bash
# Generate SBOM
syft myapp:v1.0.0 -o spdx-json > sbom.json

# Sign SBOM as attestation
cosign attest --predicate sbom.json --key cosign.key myapp:v1.0.0

# Verify attestation
cosign verify-attestation --key cosign.pub myapp:v1.0.0
```

### SLSA Provenance

```bash
# Generate provenance
slsa-provenance generate \
  --artifact myapp:v1.0.0 \
  --builder github-actions \
  > provenance.json

# Attest provenance
cosign attest --predicate provenance.json --key cosign.key myapp:v1.0.0
```

---

## 🔍 Transparency Logs (Rekor)

All signatures are stored in a public, immutable transparency log.

```bash
# Search Rekor for image signatures
rekor-cli search --artifact myapp:v1.0.0

# Get signature details
rekor-cli get --uuid <uuid>
```

**Benefits**:
- Audit trail
- Detect backdating
- Public verification

---

## 🎯 Complete Workflow

```yaml
name: Secure Image Pipeline

on: [push]

permissions:
  contents: read
  packages: write
  id-token: write

jobs:
  build-sign-attest:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Install tools
        run: |
          # Cosign
          curl -sLO https://github.com/sigstore/cosign/releases/latest/download/cosign-linux-amd64
          sudo mv cosign-linux-amd64 /usr/local/bin/cosign
          sudo chmod +x /usr/local/bin/cosign
          
          # Syft (SBOM)
          curl -sSfL https://raw.githubusercontent.com/anchore/syft/main/install.sh | sh -s -- -b /usr/local/bin
          
      - name: Build and push
        run: |
          docker build -t ghcr.io/${{ github.repository }}:${{ github.sha }} .
          docker push ghcr.io/${{ github.repository }}:${{ github.sha }}
          
      - name: Generate SBOM
        run: |
          syft ghcr.io/${{ github.repository }}:${{ github.sha }} -o spdx-json > sbom.json
          
      - name: Sign image
        run: |
          cosign sign ghcr.io/${{ github.repository }}:${{ github.sha }}
          
      - name: Attest SBOM
        run: |
          cosign attest --predicate sbom.json ghcr.io/${{ github.repository }}:${{ github.sha }}
          
      - name: Verify
        run: |
          cosign verify \
            --certificate-identity=${{ github.actor }}@users.noreply.github.com \
            --certificate-oidc-issuer=https://token.actions.githubusercontent.com \
            ghcr.io/${{ github.repository }}:${{ github.sha }}
```

---

## ✅ Best Practices

- [ ] Use keyless signing (OIDC) when possible
- [ ] Store private keys in secrets manager
- [ ] Rotate keys regularly
- [ ] Sign all production images
- [ ] Enforce signature verification in Kubernetes
- [ ] Attest SBOMs and provenance
- [ ] Monitor Rekor for your images
- [ ] Document your signing process

---

## 📚 Tools

- **Cosign**: Image signing (Sigstore)
- **Rekor**: Transparency log
- **Kyverno**: Policy enforcement
- **Syft**: SBOM generation
- **Notary**: Alternative signing (deprecated)

---

**Next**: Learn about [Supply Chain Security](./supply-chain-security.md) and [SBOM generation](./supply-chain-security.md#sbom).
