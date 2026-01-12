# DevSecOps Interview Questions

## 🎯 Fundamentals

**Q: What is DevSecOps?**

**A:** DevSecOps integrates security practices throughout the DevOps lifecycle, making security a shared responsibility rather than a late-stage gate. Key principles:
- Shift Left: Find issues early
- Automate security testing
- Security as Code
- Continuous compliance

**Q: Explain "Shift Left" in security.**

**A:** Moving security earlier in the development lifecycle:
- Pre-commit hooks for secrets
- SAST in pull requests
- SCA in CI pipeline
- Security reviews during design

## 🔍 Security Testing

**Q: Explain SAST, DAST, and SCA.**

**A:**
- **SAST** (Static): Analyzes source code without execution
- **DAST** (Dynamic): Tests running application
- **SCA** (Composition): Scans dependencies for vulnerabilities

**Q: What tools would you use for each?**

| Type | Tools |
|------|-------|
| SAST | Semgrep, SonarQube, CodeQL |
| DAST | OWASP ZAP, Burp Suite |
| SCA | Trivy, Snyk, Dependabot |

**Q: How do you handle false positives in security scanning?**

**A:**
1. Triage and document reasoning
2. Add to ignore list with expiration
3. Create custom rules to improve accuracy
4. Regular review of suppressions
5. Track metrics on false positive rates

## 🔐 Secrets Management

**Q: How do you prevent secrets in source code?**

**A:**
- Pre-commit hooks (gitleaks, trufflehog)
- CI pipeline scanning
- GitHub secret scanning
- Developer education
- Git history cleanup for exposed secrets

**Q: Compare Vault, AWS Secrets Manager, and SOPS.**

| Feature | Vault | Secrets Manager | SOPS |
|---------|-------|-----------------|------|
| Dynamic secrets | Yes | Limited | No |
| GitOps friendly | No | No | Yes |
| Self-managed | Yes | No | Partial |
| Cost | Infrastructure | Per secret | Free |

## 📦 Container Security

**Q: How do you secure container images?**

**A:**
1. Use minimal base images (distroless, Alpine)
2. Don't run as root
3. Scan images for vulnerabilities
4. Sign images with Cosign
5. Use read-only root filesystem
6. Drop all capabilities

**Q: Explain Kubernetes security best practices.**

**A:**
- Enable RBAC, disable anonymous auth
- Use Pod Security Standards
- Implement Network Policies
- Enable audit logging
- Use admission controllers (OPA, Kyverno)
- Encrypt secrets at rest

## 🔗 Supply Chain Security

**Q: What is SLSA and why does it matter?**

**A:** Supply chain Levels for Software Artifacts - framework for supply chain integrity:
- Level 1: Build process documented
- Level 2: Hosted build, signed provenance
- Level 3: Hardened build platform
- Level 4: Hermetic, reproducible builds

**Q: How do you implement supply chain security?**

**A:**
- Pin and verify dependencies
- Generate and verify SBOMs
- Sign artifacts with Sigstore/Cosign
- Use attestations for provenance
- Verify signatures at deployment

## 🎯 Scenario Questions

**Q: A developer accidentally committed an API key. What do you do?**

**A:**
1. **Immediate**: Rotate/revoke the key
2. **Remove**: Use git-filter-repo to remove from history
3. **Notify**: Alert affected parties
4. **Prevent**: Add pre-commit hooks
5. **Document**: Create incident report

**Q: Design a secure CI/CD pipeline.**

**A:**
```
┌─────────────────────────────────────────┐
│ Pre-commit: Secrets scan, linting      │
└──────────────────┬──────────────────────┘
                   ▼
┌─────────────────────────────────────────┐
│ Build: SAST, SCA, Unit tests           │
└──────────────────┬──────────────────────┘
                   ▼
┌─────────────────────────────────────────┐
│ Test: DAST, Integration, Security      │
└──────────────────┬──────────────────────┘
                   ▼
┌─────────────────────────────────────────┐
│ Release: Image scan, sign, SBOM        │
└──────────────────┬──────────────────────┘
                   ▼
┌─────────────────────────────────────────┐
│ Deploy: Verify signature, admission    │
└─────────────────────────────────────────┘
```

---

**Next**: Review [AI Security Interview](./ai-security.md) questions.

