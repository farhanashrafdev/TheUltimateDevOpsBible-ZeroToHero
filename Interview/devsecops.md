# DevSecOps Interview Questions

## 🎯 Introduction
DevSecOps integrates security into every stage of the DevOps lifecycle ("Shifting Left").

## 🟢 Concepts

### 1. What does "Shift Left" mean?
**Answer**: Moving security checks earlier in the SDLC (e.g., during coding/build) rather than waiting for a security audit before deployment. Cheaper and faster to fix bugs.

### 2. Explain the "CIA Triad".
**Answer**:
- **Confidentiality**: Only authorized users can access data.
- **Integrity**: Data is not tampered with.
- **Availability**: System is accessible when needed.

### 3. What is SAST vs. DAST?
**Answer**:
- **SAST (Static Application Security Testing)**: Scans source code (white-box). Finds SQLi, buffer overflows. (e.g., SonarQube).
- **DAST (Dynamic Application Security Testing)**: Scans running application (black-box). Finds runtime issues. (e.g., OWASP ZAP).

---

## 🟡 Implementation

### 4. How do you secure a Docker container?
**Answer**:
- Use minimal base images (Distroless/Alpine).
- Run as non-root user.
- Scan image for vulnerabilities (Trivy).
- Sign image (Cosign).
- Read-only filesystem.
- Limit resources (CPU/RAM).
- Drop capabilities (`ALL`, add back `NET_BIND_SERVICE`).

### 5. How do you manage secrets in Kubernetes?
**Answer**:
- **Native Secrets**: Base64 encoded (not encrypted!). Enable Encryption at Rest (etcd).
- **External Secrets Operator**: Sync secrets from Vault/AWS Secrets Manager.
- **Sealed Secrets**: Encrypt secrets in Git, decrypt in cluster.
- **CSI Driver**: Mount secrets as volumes from Vault.

### 6. Explain "Supply Chain Security".
**Answer**: Securing all components that go into your software (dependencies, build tools).
- **SBOM**: Software Bill of Materials.
- **Signing**: Verify origin of artifacts.
- **SLSA**: Framework for build integrity.

---

## 🔴 Advanced

### 7. How would you implement "Zero Trust" in a Kubernetes cluster?
**Answer**:
- **Identity**: Every pod has an identity (ServiceAccount).
- **mTLS**: Encrypt all pod-to-pod traffic (Istio/Linkerd).
- **Network Policies**: Deny all traffic by default. Allowlist specific paths.
- **Authentication**: OIDC for users.

### 8. You found a critical CVE in a production library. What is your response plan?
**Answer**:
1. **Assess Impact**: Is the vulnerable function used? Is it exposed?
2. **Patch**: Update library in dev.
3. **Test**: Run regression tests.
4. **Deploy**: Fast-track deployment to prod.
5. **Mitigate**: If patch not available, use WAF rules to block exploit.

### 9. Explain "Least Privilege" in AWS IAM.
**Answer**: Granting only the permissions necessary to perform a task.
- Use `AccessAnalyzer` to find unused permissions.
- Use `IAM Roles` instead of Users.
- Use `Conditions` (e.g., only from specific IP).

---

## 🧠 Scenario

### 10. Design a secure CI/CD pipeline.
**Answer**:
1. **Code**: Branch protection, signed commits.
2. **Pre-Build**: Secret scanning (TruffleHog).
3. **Build**: SAST scan, SCA (Dependency) scan.
4. **Artifact**: Container scan (Trivy), Image Signing (Cosign).
5. **Deploy**: Verify signature, OPA Gatekeeper policy check.
