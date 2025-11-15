# Security Cheat Sheet

## 📚 Scanning Tools

### Trivy
```bash
trivy image nginx     # Scan image
trivy fs .           # Scan filesystem
trivy k8s cluster     # Scan cluster
```

### Checkov
```bash
checkov -d terraform/  # Scan Terraform
checkov -f file.yaml   # Scan file
```

### Snyk
```bash
snyk test            # Test dependencies
snyk monitor         # Monitor project
snyk container test image  # Test container
```

## 📚 Vault
```bash
vault kv get secret/app  # Get secret
vault kv put secret/app key=value  # Put secret
vault auth -method=userpass  # Authenticate
```

---

**Quick Reference**: Essential security tools.

