# SAST, SCA, DAST

## 🎯 Security Testing Types

### SAST (Static Application Security Testing)
- Analyzes source code
- Finds vulnerabilities early
- No running application needed
- Examples: SonarQube, Checkmarx, Veracode

### SCA (Software Composition Analysis)
- Scans dependencies
- Finds known vulnerabilities
- License compliance
- Examples: Snyk, WhiteSource, Dependabot

### DAST (Dynamic Application Security Testing)
- Tests running applications
- Finds runtime vulnerabilities
- Simulates attacks
- Examples: OWASP ZAP, Burp Suite

## 📝 Tool Examples

### SonarQube (SAST)
```yaml
- name: SonarQube Scan
  uses: sonarsource/sonarqube-scan-action@master
  env:
    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

### Snyk (SCA)
```yaml
- name: Snyk Test
  uses: snyk/actions/node@master
  env:
    SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
```

### OWASP ZAP (DAST)
```yaml
- name: ZAP Scan
  uses: zaproxy/action-baseline@v0.7.0
  with:
    target: 'https://example.com'
```

## ✅ Best Practices

- Use all three types
- Integrate in CI/CD
- Regular scans
- Fix critical issues
- Track remediation

---

**Next**: Learn Infrastructure as Code scanning.

