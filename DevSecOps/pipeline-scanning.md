# Pipeline Scanning

## 🎯 Automated Security Scanning

Integrate security scanning into CI/CD pipelines for continuous security validation.

## 🔍 Scan Types

### Code Scanning
- SAST tools
- Secret detection
- Code quality

### Dependency Scanning
- SCA tools
- License compliance
- Vulnerability detection

### Infrastructure Scanning
- IaC security
- Misconfiguration detection
- Policy validation

### Container Scanning
- Image vulnerabilities
- Base image issues
- Configuration problems

## 📝 Implementation

### GitHub Actions Example
```yaml
- name: Run SAST
  uses: github/super-linter@v4
  env:
    DEFAULT_BRANCH: main

- name: Dependency Scan
  uses: snyk/actions/node@master
  env:
    SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
```

### GitLab CI Example
```yaml
sast:
  stage: test
  include:
    - template: Security/SAST.gitlab-ci.yml
```

## ✅ Best Practices

- Scan early and often
- Fail on critical issues
- Automate remediation
- Track vulnerabilities
- Regular updates

---

**Next**: Learn SAST, SCA, and DAST tools.

