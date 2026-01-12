# KMS, Vault, and SOPS: Secrets Management in Practice

## 🎯 Introduction

This guide covers the three primary approaches to secrets management in modern infrastructure: cloud-native KMS services, HashiCorp Vault, and Mozilla SOPS for GitOps workflows.

## 📚 AWS KMS (Key Management Service)

### Core Concepts

```
┌─────────────────────────────────────────────────────────────────────┐
│                         AWS KMS Architecture                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Customer Master Key (CMK)                                           │
│  ├── AWS Managed Keys (aws/service)                                 │
│  ├── Customer Managed Keys (you control)                            │
│  └── Customer Owned Keys (CloudHSM)                                 │
│                                                                      │
│  Key Operations:                                                     │
│  ├── Encrypt: Plaintext → Ciphertext (with CMK)                    │
│  ├── Decrypt: Ciphertext → Plaintext (with CMK)                    │
│  └── GenerateDataKey: Creates envelope encryption key               │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Creating and Using KMS Keys

```bash
# Create a KMS key
aws kms create-key \
  --description "Production secrets encryption key" \
  --key-usage ENCRYPT_DECRYPT \
  --origin AWS_KMS

# Create an alias
aws kms create-alias \
  --alias-name alias/production-secrets \
  --target-key-id <key-id>

# Encrypt data
aws kms encrypt \
  --key-id alias/production-secrets \
  --plaintext "my-secret-value" \
  --output text --query CiphertextBlob | base64 --decode > secret.enc

# Decrypt data
aws kms decrypt \
  --ciphertext-blob fileb://secret.enc \
  --output text --query Plaintext | base64 --decode
```

### KMS Key Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Enable IAM policies",
      "Effect": "Allow",
      "Principal": {"AWS": "arn:aws:iam::123456789012:root"},
      "Action": "kms:*",
      "Resource": "*"
    },
    {
      "Sid": "Allow encryption by CI/CD",
      "Effect": "Allow",
      "Principal": {"AWS": "arn:aws:iam::123456789012:role/GitHubActionsRole"},
      "Action": ["kms:Encrypt", "kms:GenerateDataKey"],
      "Resource": "*"
    },
    {
      "Sid": "Allow decryption by applications",
      "Effect": "Allow",
      "Principal": {"AWS": "arn:aws:iam::123456789012:role/ApplicationRole"},
      "Action": "kms:Decrypt",
      "Resource": "*"
    }
  ]
}
```

### Terraform KMS Configuration

```hcl
resource "aws_kms_key" "secrets" {
  description             = "Key for encrypting application secrets"
  deletion_window_in_days = 30
  enable_key_rotation     = true
  
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid       = "Enable IAM policies"
        Effect    = "Allow"
        Principal = { AWS = "arn:aws:iam::${data.aws_caller_identity.current.account_id}:root" }
        Action    = "kms:*"
        Resource  = "*"
      }
    ]
  })
  
  tags = {
    Environment = "production"
    Purpose     = "secrets-encryption"
  }
}

resource "aws_kms_alias" "secrets" {
  name          = "alias/production-secrets"
  target_key_id = aws_kms_key.secrets.key_id
}
```

## 🔐 HashiCorp Vault

### Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                       Vault Architecture                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Clients ──► API ──► Core ──► Barrier (Encryption) ──► Storage     │
│                        │                                             │
│                        ├── Auth Methods (OIDC, K8s, AWS, etc.)      │
│                        ├── Secrets Engines (KV, PKI, AWS, etc.)     │
│                        ├── Audit Devices (File, Syslog, Socket)     │
│                        └── Policies (ACL rules)                      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Vault Installation (Kubernetes)

```yaml
# vault-values.yaml (Helm)
server:
  ha:
    enabled: true
    replicas: 3
    raft:
      enabled: true
  
  dataStorage:
    enabled: true
    size: 10Gi
    storageClass: gp3
    
  auditStorage:
    enabled: true
    size: 10Gi
    
  ingress:
    enabled: true
    hosts:
      - host: vault.company.com
        
injector:
  enabled: true
  
ui:
  enabled: true
```

```bash
helm install vault hashicorp/vault -f vault-values.yaml -n vault
```

### Vault Configuration

```bash
# Initialize Vault
vault operator init -key-shares=5 -key-threshold=3

# Unseal (repeat with 3 different keys)
vault operator unseal <key>

# Enable secrets engine
vault secrets enable -path=secret kv-v2

# Enable Kubernetes auth
vault auth enable kubernetes

# Configure Kubernetes auth
vault write auth/kubernetes/config \
  kubernetes_host="https://kubernetes.default.svc" \
  token_reviewer_jwt="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
  kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
```

### Creating Secrets and Policies

```bash
# Create secret
vault kv put secret/production/database \
  username="admin" \
  password="supersecret123"

# Create policy
vault policy write app-read - <<EOF
path "secret/data/production/*" {
  capabilities = ["read"]
}
EOF

# Create Kubernetes role
vault write auth/kubernetes/role/app \
  bound_service_account_names=app \
  bound_service_account_namespaces=production \
  policies=app-read \
  ttl=1h
```

### Vault Agent Injector

```yaml
# deployment.yaml with Vault annotations
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  template:
    metadata:
      annotations:
        vault.hashicorp.com/agent-inject: "true"
        vault.hashicorp.com/role: "app"
        vault.hashicorp.com/agent-inject-secret-db: "secret/data/production/database"
        vault.hashicorp.com/agent-inject-template-db: |
          {{- with secret "secret/data/production/database" -}}
          DATABASE_URL=postgresql://{{ .Data.data.username }}:{{ .Data.data.password }}@db:5432/app
          {{- end -}}
    spec:
      serviceAccountName: app
      containers:
        - name: app
          image: myapp:latest
          command: ["sh", "-c", "source /vault/secrets/db && ./start.sh"]
```

### External Secrets Operator

```yaml
# ClusterSecretStore for Vault
apiVersion: external-secrets.io/v1beta1
kind: ClusterSecretStore
metadata:
  name: vault-backend
spec:
  provider:
    vault:
      server: "https://vault.company.com"
      path: "secret"
      version: "v2"
      auth:
        kubernetes:
          mountPath: "kubernetes"
          role: "external-secrets"
          serviceAccountRef:
            name: "external-secrets"
            namespace: "external-secrets"
---
# ExternalSecret
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: database-credentials
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault-backend
    kind: ClusterSecretStore
  target:
    name: database-secret
  data:
    - secretKey: username
      remoteRef:
        key: production/database
        property: username
    - secretKey: password
      remoteRef:
        key: production/database
        property: password
```

## 🔒 SOPS (Secrets OPerationS)

### Why SOPS?

SOPS encrypts files while keeping the structure visible, perfect for GitOps:

```yaml
# Encrypted SOPS file (structure visible, values encrypted)
database:
    username: ENC[AES256_GCM,data:abc123,iv:xyz,tag:def]
    password: ENC[AES256_GCM,data:ghi789,iv:uvw,tag:jkl]
sops:
    kms:
        - arn: arn:aws:kms:us-east-1:123456789:key/abc-123
    lastmodified: 2024-01-15T10:00:00Z
```

### SOPS with AWS KMS

```bash
# Install SOPS
brew install sops  # or download from GitHub

# Create .sops.yaml configuration
cat > .sops.yaml <<EOF
creation_rules:
  - path_regex: secrets/production/.*\.yaml$
    kms: arn:aws:kms:us-east-1:123456789:key/prod-key
    
  - path_regex: secrets/staging/.*\.yaml$
    kms: arn:aws:kms:us-east-1:123456789:key/staging-key
    
  - path_regex: .*\.yaml$
    kms: arn:aws:kms:us-east-1:123456789:key/default-key
EOF

# Encrypt a file
sops -e secrets.yaml > secrets.enc.yaml

# Decrypt a file
sops -d secrets.enc.yaml > secrets.yaml

# Edit encrypted file in-place
sops secrets.enc.yaml
```

### SOPS with Age

```bash
# Generate age key pair
age-keygen -o key.txt

# Configure SOPS
cat > .sops.yaml <<EOF
creation_rules:
  - age: age1ql3z7hjy54pw3hyww5ayyfg7zqgvc7w3j2elw8zmrj2kg5sfn9aqmcac8p
EOF

# Set private key
export SOPS_AGE_KEY_FILE=~/.sops/key.txt

# Encrypt
sops -e secrets.yaml > secrets.enc.yaml
```

### ArgoCD + SOPS Integration

```yaml
# argocd-cm ConfigMap for SOPS plugin
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
data:
  configManagementPlugins: |
    - name: sops
      generate:
        command: ["sh", "-c"]
        args: ["sops -d secrets.enc.yaml | kubectl kustomize"]
---
# Application using SOPS plugin
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
spec:
  source:
    plugin:
      name: sops
```

### Flux + SOPS Integration

```yaml
# Flux Kustomization with SOPS
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: app
  namespace: flux-system
spec:
  interval: 10m
  path: ./kubernetes
  prune: true
  sourceRef:
    kind: GitRepository
    name: app
  decryption:
    provider: sops
    secretRef:
      name: sops-age  # Contains the age private key
```

## 📊 Comparison

| Feature | AWS KMS | Vault | SOPS |
|---------|---------|-------|------|
| Secret Storage | AWS services | Vault | Git |
| Dynamic Secrets | Via Secrets Manager | Yes | No |
| Key Rotation | Automatic | Manual/Auto | Manual |
| Access Control | IAM | ACL Policies | File-level |
| Audit Logging | CloudTrail | Built-in | Git history |
| Best For | AWS-native | Hybrid/multi-cloud | GitOps |

## ✅ Best Practices

1. **Key Rotation**: Enable automatic key rotation
2. **Least Privilege**: Minimal permissions for each role
3. **Audit Logging**: Enable comprehensive audit trails
4. **Encryption at Rest**: Always encrypt stored secrets
5. **Version Secrets**: Keep history for rollback
6. **Separate by Environment**: Different keys per environment

---

**Next**: Learn about [Container Security](./container-security.md).

