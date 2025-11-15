# Security Labs

## 🎯 Security Practical Exercises

### Lab 1: SAST Scanning
**Objective**: Static code analysis

**Tasks**:
1. Set up SonarQube
2. Scan codebase
3. Review findings
4. Fix vulnerabilities

**Commands**:
```bash
docker run -d -p 9000:9000 sonarqube
sonar-scanner -Dsonar.projectKey=myapp
```

### Lab 2: Dependency Scanning
**Objective**: Scan dependencies

**Tasks**:
1. Install Snyk
2. Scan dependencies
3. Review vulnerabilities
4. Update dependencies

**Commands**:
```bash
npm install -g snyk
snyk test
snyk monitor
```

### Lab 3: Container Scanning
**Objective**: Scan container images

**Tasks**:
1. Build image
2. Scan with Trivy
3. Review findings
4. Fix issues

**Commands**:
```bash
docker build -t myapp .
trivy image myapp
trivy fs .
```

### Lab 4: Secrets Management
**Objective**: Secure secrets

**Tasks**:
1. Set up Vault
2. Store secrets
3. Retrieve secrets
4. Rotate secrets

**Commands**:
```bash
vault server -dev
vault kv put secret/app api_key=value
vault kv get secret/app
```

### Lab 5: IaC Scanning
**Objective**: Scan infrastructure code

**Tasks**:
1. Install Checkov
2. Scan Terraform
3. Review findings
4. Fix misconfigurations

**Commands**:
```bash
pip install checkov
checkov -d terraform/
checkov -f main.tf
```

## ✅ Completion Checklist

- [ ] Completed all scans
- [ ] Fixed vulnerabilities
- [ ] Documented findings
- [ ] Implemented best practices

---

**Next**: Complete AI SecOps labs.

