# KMS, Vault, SOPS

## 🔐 Key Management Services (KMS)

### AWS KMS
- Managed encryption keys
- Integration with AWS services
- Key rotation
- Audit logging

### Usage
```python
import boto3
kms = boto3.client('kms')
response = kms.encrypt(
    KeyId='alias/my-key',
    Plaintext='sensitive-data'
)
```

## 🏦 HashiCorp Vault

### Features
- Secrets management
- Dynamic secrets
- Encryption as a service
- Access policies

### Basic Usage
```bash
# Authenticate
vault auth -method=userpass username=user

# Read secret
vault kv get secret/app

# Write secret
vault kv put secret/app api_key=value
```

## 🔒 SOPS (Secrets OPerationS)

### Encrypted File Management
```bash
# Encrypt file
sops -e -i secrets.yaml

# Edit encrypted file
sops secrets.yaml

# Decrypt for use
sops -d secrets.yaml
```

### Integration
```yaml
# .sops.yaml
creation_rules:
  - path_regex: .*\.yaml$
    kms: 'arn:aws:kms:...'
```

## ✅ Best Practices

- Use appropriate tool
- Rotate keys
- Audit access
- Limit permissions
- Document procedures

---

**Next**: Learn container security.

