# Project: Secure CI/CD Pipeline (DevSecOps)

## 🎯 Objective
Build a pipeline that enforces security at every stage: code, build, deploy, and runtime.

## 🛠️ Tech Stack
- **CI**: GitHub Actions / GitLab CI
- **Scanning**: Trivy, SonarQube, Checkov
- **Signing**: Cosign
- **Policy**: OPA / Kyverno

## 📝 Requirements

1. **Pre-Commit**:
   - Prevent secrets from being committed (pre-commit hooks).

2. **CI Pipeline**:
   - SAST (SonarQube).
   - SCA (Dependency check).
   - Container Scan (Trivy).
   - IaC Scan (Checkov).
   - Image Signing (Cosign).

3. **Admission Control**:
   - Cluster rejects unsigned images.
   - Cluster rejects privileged pods.

## 🚀 Steps

### 1. Pipeline Construction
- Create workflow with security stages.
- Fail build on "Critical" vulnerabilities.

### 2. Signing Implementation
- Generate Cosign keys.
- Sign image in CI.
- Store public key in K8s secret.

### 3. Enforcement
- Install Kyverno.
- Apply policy to verify Cosign signature.
- Try to deploy unsigned image (should fail).

## 🏆 Definition of Done
- [ ] Pipeline fails if secrets detected.
- [ ] Pipeline fails if critical CVE found.
- [ ] Image is signed and pushed.
- [ ] Kubernetes rejects unsigned images.
- [ ] SBOM is generated and attested.

---

**Next**: [AIOps Dashboard](./aiops-dashboard.md).
