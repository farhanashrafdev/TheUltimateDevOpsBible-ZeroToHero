# Secrets Management: Complete Guide

## 🎯 Introduction

Secrets management is critical for application security. This guide covers best practices, tools, and patterns for securely managing secrets in modern applications.

### What are Secrets?

**Secrets** include:
- API keys and tokens
- Database passwords
- SSH keys
- TLS certificates
- Encryption keys
- OAuth credentials
- Service account keys

### Secrets Management Principles

1. **Never Commit Secrets**: Never store secrets in code or version control
2. **Encrypt at Rest**: All secrets encrypted when stored
3. **Encrypt in Transit**: All secrets encrypted when transmitted
4. **Rotate Regularly**: Change secrets periodically
5. **Audit Access**: Log all secret access
6. **Least Privilege**: Grant minimal access needed
7. **Centralized Management**: Use secret managers

## 🏦 HashiCorp Vault

### Installation

```bash
# Download Vault
wget https://releases.hashicorp.com/vault/1.15.0/vault_1.15.0_linux_amd64.zip
unzip vault_1.15.0_linux_amd64.zip
sudo mv vault /usr/local/bin/

# Start Vault (dev mode)
vault server -dev

# Set environment
export VAULT_ADDR='http://127.0.0.1:8200'
export VAULT_TOKEN='dev-token'
```

### Kubernetes Installation

```yaml
# Install Vault via Helm
helm repo add hashicorp https://helm.releases.hashicorp.com
helm install vault hashicorp/vault

# Initialize Vault
kubectl exec -it vault-0 -- vault operator init

# Unseal Vault
kubectl exec -it vault-0 -- vault operator unseal <key>
```

### Basic Operations

```bash
# Enable KV secrets engine
vault secrets enable -path=secret kv-v2

# Write secret
vault kv put secret/app api_key=abc123 database_password=secretpass

# Read secret
vault kv get secret/app

# List secrets
vault kv list secret/

# Delete secret
vault kv delete secret/app
```

### Dynamic Secrets

```bash
# Enable database secrets engine
vault secrets enable database

# Configure database
vault write database/config/postgresql \
  plugin_name=postgresql-database-plugin \
  allowed_roles="readonly" \
  connection_url="postgresql://{{username}}:{{password}}@postgres:5432/mydb?sslmode=disable" \
  username="vault" \
  password="vaultpass"

# Create role
vault write database/roles/readonly \
  db_name=postgresql \
  creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; GRANT SELECT ON ALL TABLES IN SCHEMA public TO \"{{name}}\";" \
  default_ttl="1h" \
  max_ttl="24h"

# Generate dynamic credentials
vault read database/creds/readonly
```

### Policies

```hcl
# app-policy.hcl
path "secret/data/app/*" {
  capabilities = ["read"]
}

path "secret/data/app/config" {
  capabilities = ["read", "create", "update"]
}

# Apply policy
vault policy write app-policy app-policy.hcl

# Create token with policy
vault token create -policy=app-policy
```

### Authentication Methods

#### AppRole

```bash
# Enable AppRole
vault auth enable approle

# Create role
vault write auth/approle/role/myapp \
  token_policies="app-policy" \
  token_ttl=1h \
  token_max_ttl=4h

# Get role ID
vault read auth/approle/role/myapp/role-id

# Get secret ID
vault write -f auth/approle/role/myapp/secret-id
```

#### Kubernetes Auth

```bash
# Enable Kubernetes auth
vault auth enable kubernetes

# Configure
vault write auth/kubernetes/config \
  token_reviewer_jwt="<service-account-jwt>" \
  kubernetes_host="https://kubernetes:443" \
  kubernetes_ca_cert=@ca.crt

# Create role
vault write auth/kubernetes/role/myapp \
  bound_service_account_names=myapp \
  bound_service_account_namespaces=production \
  policies=app-policy \
  ttl=1h
```

### Vault Agent

```yaml
# vault-agent-config.hcl
pid_file = "/tmp/vault-agent.pid"

vault {
  address = "http://vault:8200"
}

auto_auth {
  method "kubernetes" {
    mount_path = "auth/kubernetes"
    config = {
      role = "myapp"
    }
  }
  
  sink "file" {
    config = {
      path = "/tmp/vault-token"
    }
  }
}

template {
  source      = "/vault/secrets/app.tpl"
  destination = "/app/secrets/app.env"
  perms       = 0600
}
```

## ☁️ Cloud Secret Managers

### AWS Secrets Manager

```python
import boto3
import json

# Create client
secrets_client = boto3.client('secretsmanager', region_name='us-east-1')

# Store secret
secret = {
    'username': 'admin',
    'password': 'secret123'
}
secrets_client.create_secret(
    Name='app/database',
    SecretString=json.dumps(secret)
)

# Retrieve secret
response = secrets_client.get_secret_value(SecretId='app/database')
secret = json.loads(response['SecretString'])
print(secret['password'])

# Rotate secret
secrets_client.rotate_secret(SecretId='app/database')
```

**Terraform**:
```hcl
resource "aws_secretsmanager_secret" "db_password" {
  name                    = "app/database"
  recovery_window_in_days = 7
}

resource "aws_secretsmanager_secret_version" "db_password" {
  secret_id = aws_secretsmanager_secret.db_password.id
  secret_string = jsonencode({
    username = "admin"
    password = var.db_password
  })
}
```

### Azure Key Vault

```python
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient

# Authenticate
credential = DefaultAzureCredential()
client = SecretClient(vault_url="https://myvault.vault.azure.net/", credential=credential)

# Set secret
client.set_secret("database-password", "secret123")

# Get secret
secret = client.get_secret("database-password")
print(secret.value)

# Delete secret
client.begin_delete_secret("database-password")
```

### Google Secret Manager

```python
from google.cloud import secretmanager

# Create client
client = secretmanager.SecretManagerServiceClient()

# Create secret
project_id = "my-project"
secret_id = "database-password"
parent = f"projects/{project_id}"
client.create_secret(request={"parent": parent, "secret_id": secret_id, "secret": {"replication": {"automatic": {}}}})

# Add secret version
name = f"projects/{project_id}/secrets/{secret_id}"
payload = "secret123".encode("UTF-8")
client.add_secret_version(request={"parent": name, "payload": {"data": payload}})

# Access secret
name = f"projects/{project_id}/secrets/{secret_id}/versions/latest"
response = client.access_secret_version(request={"name": name})
print(response.payload.data.decode("UTF-8"))
```

## ☸️ Kubernetes Secrets

### Native Secrets

```yaml
# Create secret
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  username: YWRtaW4=  # base64 encoded
  password: c2VjcmV0MTIz
stringData:
  api_key: "secret-key"  # Automatically encoded
```

### Using Secrets

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp:latest
    env:
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: username
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: password
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: app-secret
```

### External Secrets Operator

```yaml
# Install External Secrets Operator
kubectl apply -f https://raw.githubusercontent.com/external-secrets/external-secrets/main/deploy/charts/external-secrets/templates/crds/secretstore.yaml

# Create SecretStore
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: vault-backend
spec:
  provider:
    vault:
      server: "https://vault.example.com"
      path: "secret"
      version: "v2"
      auth:
        kubernetes:
          mountPath: "kubernetes"
          role: "myapp"

# Create ExternalSecret
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: app-secret
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault-backend
    kind: SecretStore
  target:
    name: app-secret
    creationPolicy: Owner
  data:
  - secretKey: password
    remoteRef:
      key: secret/app
      property: database_password
```

## 🔒 SOPS (Secrets OPerationS)

### Installation

```bash
# Install SOPS
brew install sops
# or
wget https://github.com/mozilla/sops/releases/download/v3.8.0/sops-v3.8.0.linux
chmod +x sops-v3.8.0.linux
sudo mv sops-v3.8.0.linux /usr/local/bin/sops
```

### Configuration

```yaml
# .sops.yaml
creation_rules:
  - path_regex: \.yaml$
    kms: 'arn:aws:kms:us-east-1:123456789012:key/abc123'
    pgp: 'FINGERPRINT'
  - path_regex: \.env$
    kms: 'arn:aws:kms:us-east-1:123456789012:key/abc123'
```

### Usage

```bash
# Encrypt file
sops -e -i secrets.yaml

# Edit encrypted file
sops secrets.yaml

# Decrypt file
sops -d secrets.yaml

# Encrypt with AWS KMS
sops -e -k arn:aws:kms:us-east-1:123456789012:key/abc123 secrets.yaml

# Encrypt with PGP
sops -e -pgp FINGERPRINT secrets.yaml
```

### CI/CD Integration

```yaml
# GitHub Actions
- name: Decrypt secrets
  run: |
    sops -d secrets.yaml > decrypted.yaml
  env:
    AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
    AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

## 🔄 Secret Rotation

### Automated Rotation

```python
# Rotate secret
import boto3
import secrets
import string

def rotate_secret(secret_id):
    client = boto3.client('secretsmanager')
    
    # Generate new password
    new_password = ''.join(secrets.choice(string.ascii_letters + string.digits) for _ in range(32))
    
    # Update secret
    client.update_secret(
        SecretId=secret_id,
        SecretString=json.dumps({'password': new_password})
    )
    
    # Update in application
    update_database_password(new_password)
```

### Rotation Lambda (AWS)

```python
import boto3
import json

def lambda_handler(event, context):
    secrets_client = boto3.client('secretsmanager')
    
    # Get secret
    secret = secrets_client.get_secret_value(SecretId=event['SecretId'])
    current_secret = json.loads(secret['SecretString'])
    
    # Generate new password
    new_password = generate_password()
    
    # Update secret
    secrets_client.update_secret(
        SecretId=event['SecretId'],
        SecretString=json.dumps({
            'username': current_secret['username'],
            'password': new_password
        })
    )
    
    # Update database
    update_database_password(new_password)
    
    return {'statusCode': 200}
```

## ✅ Best Practices

### 1. Never Commit Secrets

```yaml
# .gitignore
*.key
*.pem
*.p12
secrets.yaml
.env
*.secret
```

### 2. Use Secret Managers

```yaml
# Don't hardcode
# Use Vault, AWS Secrets Manager, etc.
# Centralized management
```

### 3. Rotate Regularly

```yaml
# Automated rotation
# Regular schedule
# After incidents
# Key rotation policies
```

### 4. Audit Access

```yaml
# Log all access
# Monitor usage
# Alert on anomalies
# Regular reviews
```

### 5. Least Privilege

```yaml
# Minimal access
# Role-based access
# Time-limited access
# Just-in-time access
```

## ✅ Mastery Checklist

- [ ] Understand secrets management principles
- [ ] Set up HashiCorp Vault
- [ ] Use cloud secret managers
- [ ] Manage Kubernetes secrets
- [ ] Implement secret rotation
- [ ] Audit secret access
- [ ] Use SOPS for encrypted files
- [ ] Integrate in CI/CD
- [ ] Monitor secret usage
- [ ] Follow best practices

---

**Next Steps**:
- Learn [KMS, Vault, SOPS](./kms-vault-sops.md)
- Explore [CI/CD Security](./ci-cd-security.md)
- Master [Container Security](./container-security.md)

**Remember**: Secrets management is critical for security. Never commit secrets, use secret managers, rotate regularly, and audit access. Start with basics, implement proper tools, and continuously improve your secrets management practices.
