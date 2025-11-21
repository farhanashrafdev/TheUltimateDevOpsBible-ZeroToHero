# KMS, Vault, SOPS: Secrets Management Solutions

## 🎯 Introduction

Three popular secrets management solutions compared.

## 🔐 Solutions

### 1. AWS KMS (Key Management Service)

**Use Case**: AWS-native encryption

```bash
# Encrypt
aws kms encrypt \
  --key-id alias/my-key \
  --plaintext "secret" \
  --output text \
  --query CiphertextBlob

# Decrypt
aws kms decrypt \
  --ciphertext-blob fileb://encrypted.txt \
  --output text \
  --query Plaintext | base64 --decode
```

### 2. HashiCorp Vault

**Use Case**: Centralized secrets management

```bash
# Start Vault
vault server -dev

# Store secret
vault kv put secret/myapp password=supersecret

# Retrieve secret
vault kv get secret/myapp

# Dynamic secrets (AWS)
vault read aws/creds/my-role
```

**Kubernetes Integration**:
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: myapp
  annotations:
    vault.hashicorp.com/agent-inject: "true"
    vault.hashicorp.com/role: "myapp"
    vault.hashicorp.com/agent-inject-secret-db: "database/creds/myapp"
```

### 3. SOPS (Secrets OPerationS)

**Use Case**: Encrypted files in Git

```bash
# Install
brew install sops

# Encrypt file
sops -e secrets.yaml > secrets.enc.yaml

# Decrypt file
sops -d secrets.enc.yaml

# Edit encrypted file
sops secrets.enc.yaml
```

**With AWS KMS**:
```yaml
# .sops.yaml
creation_rules:
  - kms: 'arn:aws:kms:us-east-1:123456789:key/abc-123'
```

---

## 📊 Comparison

| Feature | AWS KMS | Vault | SOPS |
|:---|:---|:---|:---|
| **Complexity** | Low | High | Low |
| **Cost** | Pay per use | Self-hosted | Free |
| **Dynamic Secrets** | No | Yes | No |
| **Audit Log** | CloudTrail | Built-in | No |
| **Best For** | AWS-only | Enterprise | GitOps |

---

## 🎯 Recommendations

- **AWS-only**: Use AWS KMS
- **Multi-cloud/Enterprise**: Use Vault
- **GitOps**: Use SOPS

---

**Next**: Complete [DevSecOps Overview](./overview.md).
