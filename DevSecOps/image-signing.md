# Image Signing and Verification

## 🎯 Introduction

Image signing cryptographically proves that container images come from trusted sources and haven't been tampered with. This is critical for supply chain security.

## 📚 Why Sign Images?

```
┌─────────────────────────────────────────────────────────────────────┐
│                  Container Supply Chain Attacks                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Attack Vectors:                                                     │
│  ├── Compromised base images                                        │
│  ├── Malicious dependencies injected during build                   │
│  ├── Man-in-the-middle attacks during image pull                   │
│  ├── Typosquatting (malicious similar-named images)                │
│  └── Registry compromise                                            │
│                                                                      │
│  Image Signing Provides:                                             │
│  ├── Authenticity: Image comes from trusted source                  │
│  ├── Integrity: Image hasn't been modified                          │
│  └── Non-repudiation: Publisher can't deny publishing               │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 🔐 Cosign (Sigstore)

### Installation

```bash
# macOS
brew install cosign

# Linux
curl -LO https://github.com/sigstore/cosign/releases/latest/download/cosign-linux-amd64
chmod +x cosign-linux-amd64
sudo mv cosign-linux-amd64 /usr/local/bin/cosign
```

### Keyless Signing (Recommended)

```bash
# Sign image with OIDC identity (keyless)
cosign sign ghcr.io/myorg/myapp:v1.0.0

# This will:
# 1. Authenticate via OIDC (GitHub, Google, etc.)
# 2. Get short-lived certificate from Fulcio
# 3. Sign the image
# 4. Store signature in Rekor transparency log
```

### Key-based Signing

```bash
# Generate key pair
cosign generate-key-pair

# Sign image
cosign sign --key cosign.key ghcr.io/myorg/myapp:v1.0.0

# Verify image
cosign verify --key cosign.pub ghcr.io/myorg/myapp:v1.0.0
```

### GitHub Actions Integration

```yaml
# .github/workflows/build-sign.yml
name: Build and Sign

on:
  push:
    tags: ['v*']

jobs:
  build-sign:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      id-token: write  # Required for keyless signing

    steps:
      - uses: actions/checkout@v4

      - name: Install Cosign
        uses: sigstore/cosign-installer@v3

      - name: Login to GHCR
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and Push
        id: build
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ github.ref_name }}

      - name: Sign Image (Keyless)
        run: |
          cosign sign --yes ghcr.io/${{ github.repository }}@${{ steps.build.outputs.digest }}
        env:
          COSIGN_EXPERIMENTAL: "true"
```

### Verifying Signatures

```bash
# Verify keyless signature
cosign verify \
  --certificate-identity "https://github.com/myorg/myrepo/.github/workflows/build.yml@refs/heads/main" \
  --certificate-oidc-issuer "https://token.actions.githubusercontent.com" \
  ghcr.io/myorg/myapp:v1.0.0

# Verify with key
cosign verify --key cosign.pub ghcr.io/myorg/myapp:v1.0.0

# Output JSON for automation
cosign verify --output json ghcr.io/myorg/myapp:v1.0.0 | jq
```

## 🛡️ Kubernetes Admission Control

### Kyverno Policy

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: verify-image-signature
spec:
  validationFailureAction: Enforce
  background: false
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
            - entries:
                - keyless:
                    subject: "https://github.com/myorg/*"
                    issuer: "https://token.actions.githubusercontent.com"
                    rekor:
                      url: https://rekor.sigstore.dev
```

### Sigstore Policy Controller

```yaml
# Install policy-controller
helm install policy-controller \
  sigstore/policy-controller \
  -n cosign-system \
  --create-namespace

# Create ClusterImagePolicy
apiVersion: policy.sigstore.dev/v1beta1
kind: ClusterImagePolicy
metadata:
  name: verify-production-images
spec:
  images:
    - glob: "ghcr.io/myorg/**"
  authorities:
    - keyless:
        identities:
          - issuer: https://token.actions.githubusercontent.com
            subject: https://github.com/myorg/myrepo/.github/workflows/build.yml@refs/heads/main
        ctlog:
          url: https://rekor.sigstore.dev
```

## 📋 SBOM Attestation

### Generate and Attach SBOM

```yaml
# In GitHub Actions
- name: Generate SBOM
  uses: anchore/sbom-action@v0
  with:
    image: ghcr.io/${{ github.repository }}:${{ github.ref_name }}
    format: cyclonedx-json
    output-file: sbom.json

- name: Attest SBOM
  run: |
    cosign attest --yes \
      --predicate sbom.json \
      --type cyclonedx \
      ghcr.io/${{ github.repository }}@${{ steps.build.outputs.digest }}
```

### Verify SBOM Attestation

```bash
# Verify and extract SBOM
cosign verify-attestation \
  --type cyclonedx \
  --certificate-identity "..." \
  --certificate-oidc-issuer "..." \
  ghcr.io/myorg/myapp:v1.0.0 | jq -r '.payload' | base64 -d | jq
```

## ✅ Best Practices

1. **Use Keyless Signing**: Eliminates key management burden
2. **Enforce at Admission**: Block unsigned images in Kubernetes
3. **Sign + Attest**: Include SBOMs and build provenance
4. **Verify in CI/CD**: Check signatures before deployment
5. **Use Transparency Logs**: Rekor provides audit trail

---

**Next**: Learn about [K8s Runtime Security](./k8s-runtime-security.md).

