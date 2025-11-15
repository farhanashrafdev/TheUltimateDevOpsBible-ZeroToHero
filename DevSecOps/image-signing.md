# Image Signing

## 🎯 Container Image Signing

Sign container images to ensure integrity and authenticity.

## 🔐 Signing Methods

### Cosign
- Key-based signing
- Keyless signing
- OCI-compatible

### Notary
- TUF-based
- Content trust
- Docker integration

## 📝 Cosign Usage

### Key-based Signing
```bash
# Generate key pair
cosign generate-key-pair

# Sign image
cosign sign --key cosign.key user/image:tag

# Verify signature
cosign verify --key cosign.pub user/image:tag
```

### Keyless Signing
```bash
# Sign with OIDC
cosign sign user/image:tag

# Verify
cosign verify user/image:tag
```

## 🔧 CI/CD Integration

### GitHub Actions
```yaml
- name: Sign image
  uses: sigstore/cosign-installer@v2
- name: Sign
  run: |
    cosign sign --yes ${{ env.IMAGE }}
  env:
    COSIGN_EXPERIMENTAL: 1
```

## ✅ Best Practices

- Sign all images
- Verify before deploy
- Use keyless for CI/CD
- Store keys securely
- Automate signing
- Monitor signatures

---

**Next**: Learn zero trust architecture.

