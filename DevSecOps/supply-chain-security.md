# Supply Chain Security

## 🎯 Securing Software Supply Chain

Protect the entire software supply chain from source to deployment.

## 🔍 Security Areas

### Source Code
- Secure repositories
- Access controls
- Code signing
- Branch protection

### Dependencies
- Dependency scanning
- SBOM generation
- License compliance
- Vulnerability tracking

### Build Process
- Secure CI/CD
- Build verification
- Artifact signing
- Provenance

### Distribution
- Image signing
- Registry security
- Access controls
- Integrity verification

## 🛠️ Tools & Standards

### SBOM (Software Bill of Materials)
- SPDX format
- CycloneDX format
- Dependency tracking

### SLSA Framework
- Source integrity
- Build integrity
- Provenance
- Common requirements

### Cosign
- Container signing
- Signature verification
- Keyless signing

## 📝 Examples

### Generate SBOM
```bash
syft packages docker:nginx:latest -o spdx-json > sbom.json
```

### Sign Image
```bash
cosign sign --key cosign.key user/image:tag
cosign verify --key cosign.pub user/image:tag
```

## ✅ Best Practices

- Generate SBOMs
- Sign artifacts
- Verify signatures
- Scan dependencies
- Monitor supply chain
- Implement SLSA

---

**Next**: Learn image signing.

