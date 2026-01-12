# Secure ML Pipelines

## 🎯 Introduction

ML pipelines require security at every stage - data ingestion, training, validation, and deployment.

## 📚 Pipeline Security Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Secure ML Pipeline                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Data Ingestion      Training           Validation      Deployment  │
│  ├── Access control  ├── Isolated env   ├── Model scan  ├── Signed │
│  ├── Encryption      ├── Audit logging  ├── Bias check  ├── RBAC   │
│  ├── Validation      ├── Checkpointing  ├── Security    ├── Monitor│
│  └── Provenance      └── Secrets mgmt       tests       └── Rollback│
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 🔧 Security Controls

### Data Security

```python
# Encrypted data loading
from cryptography.fernet import Fernet

class SecureDataLoader:
    def __init__(self, key_path: str):
        with open(key_path, 'rb') as f:
            self.cipher = Fernet(f.read())
    
    def load_encrypted_dataset(self, path: str):
        with open(path, 'rb') as f:
            encrypted_data = f.read()
        
        decrypted = self.cipher.decrypt(encrypted_data)
        return self.parse_dataset(decrypted)
```

### Training Environment

```yaml
# Isolated training container
apiVersion: v1
kind: Pod
metadata:
  name: ml-training
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
  containers:
    - name: trainer
      image: ml-trainer:v1
      securityContext:
        readOnlyRootFilesystem: true
        allowPrivilegeEscalation: false
      resources:
        limits:
          nvidia.com/gpu: 1
      volumeMounts:
        - name: data
          mountPath: /data
          readOnly: true
        - name: output
          mountPath: /output
```

### Model Signing

```bash
# Sign trained model with Cosign
cosign sign-blob --key cosign.key \
  --output-signature model.sig \
  model.pkl

# Verify before deployment
cosign verify-blob --key cosign.pub \
  --signature model.sig \
  model.pkl
```

## 📝 Pipeline Example (GitHub Actions)

```yaml
name: Secure ML Pipeline

on:
  push:
    paths: ['models/**', 'data/**']

jobs:
  data-validation:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Validate Data
        run: python scripts/validate_data.py

  train:
    needs: data-validation
    runs-on: [self-hosted, gpu]
    steps:
      - uses: actions/checkout@v4
      - name: Train Model
        run: python train.py
        env:
          DATA_KEY: ${{ secrets.DATA_ENCRYPTION_KEY }}
      - name: Sign Model
        run: cosign sign-blob --key ${{ secrets.COSIGN_KEY }} model.pkl

  security-scan:
    needs: train
    steps:
      - name: Scan Model
        run: python scripts/model_security_scan.py

  deploy:
    needs: security-scan
    steps:
      - name: Verify Signature
        run: cosign verify-blob model.pkl
      - name: Deploy
        run: kubectl apply -f deployment.yaml
```

## ✅ Checklist

- [ ] Data encrypted at rest and in transit
- [ ] Training runs in isolated environment
- [ ] Models signed before deployment
- [ ] Access controls on all artifacts
- [ ] Audit logging enabled
- [ ] Vulnerability scanning on dependencies

---

**Next**: Learn about [Guardrails Architecture](./guardrails-architecture.md).

