# DevSecOps Interview Questions

## 🎯 Security Fundamentals

**Q: What is DevSecOps?**
A: Integration of security into DevOps, shifting security left, making security everyone's responsibility.

**Q: Explain SAST, SCA, DAST**
A: SAST (Static) analyzes source code. SCA (Software Composition) scans dependencies. DAST (Dynamic) tests running applications.

**Q: How do you manage secrets?**
A: Use secret managers (Vault, AWS Secrets Manager), never commit secrets, rotate regularly, implement access controls.

## 🔒 Security Practices

**Q: How do you secure containers?**
A: Scan images, use minimal base images, run as non-root, implement runtime protection, use network policies.

**Q: Explain supply chain security**
A: Secure entire software supply chain: source code, dependencies, build process, distribution. Use SBOM, signing, verification.

**Q: What is zero trust?**
A: Never trust, always verify. Authenticate all requests, least privilege access, assume breach, continuous monitoring.

## 📝 Scenario Questions

**Q: How do you implement security in CI/CD?**
A: Security gates at each stage, automated scanning (SAST/SCA/DAST), secrets management, IaC scanning, policy enforcement.

**Q: Design a secure container deployment**
A: Image scanning, signed images, runtime protection, network policies, RBAC, secrets management, monitoring.

## ✅ Key Areas

- Security scanning
- Secrets management
- Container security
- Supply chain security
- Compliance
- Incident response

---

**Next**: Review AI security interview questions.

