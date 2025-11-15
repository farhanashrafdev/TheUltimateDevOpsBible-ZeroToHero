# Secrets Management

## 🎯 Secure Secrets Handling

Proper secrets management is critical for application security.

## 🔒 Principles

- Never commit secrets
- Encrypt at rest
- Encrypt in transit
- Rotate regularly
- Audit access

## 🛠️ Tools

### HashiCorp Vault
- Centralized secrets
- Dynamic secrets
- Access policies

### AWS Secrets Manager
- Managed service
- Automatic rotation
- Integration with AWS

### Kubernetes Secrets
- Native K8s secrets
- Encrypted at rest
- RBAC controls

## 📝 Best Practices

### Environment Variables
```yaml
env:
  - name: API_KEY
    valueFrom:
      secretKeyRef:
        name: app-secrets
        key: api-key
```

### Vault Integration
```yaml
- name: Get Secret
  uses: hashicorp/vault-action@v2
  with:
    url: ${{ secrets.VAULT_ADDR }}
    method: approle
    roleId: ${{ secrets.VAULT_ROLE_ID }}
    secretId: ${{ secrets.VAULT_SECRET_ID }}
    secrets: |
      secret/data/app api_key
```

## ✅ Key Practices

- Use secret managers
- Rotate regularly
- Limit access
- Audit usage
- Never log secrets

---

**Next**: Learn KMS, Vault, and SOPS.

