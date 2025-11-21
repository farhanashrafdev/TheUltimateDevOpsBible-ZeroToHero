# Secure ML Pipelines: MLOps Security

## 🎯 Introduction

ML pipelines automate the path from data to deployment. Securing this pipeline is essential to prevent supply chain attacks and model poisoning.

## 🏗️ Secure Pipeline Architecture

```
[Data Source] → [Training] → [Model Registry] → [Deployment]
     🔒            🔒              🔒               🔒
 Encrypted      Isolated         Signed           Verified
```

---

## 🛡️ Security Controls

### 1. Data Security

- **Encryption**: Encrypt data at rest and in transit
- **Access Control**: RBAC for data lakes (S3/GCS)
- **Sanitization**: Remove PII before training

### 2. Training Security

**Isolated Environments**:
- Run training jobs in ephemeral containers
- No internet access (except specific repositories)
- Use minimal base images

**Kubeflow / SageMaker**:
- Use IAM roles for training jobs
- Enable network isolation (VPC only)

### 3. Model Registry Security

**MLflow / WandB**:
- Enforce authentication
- Track model lineage (Git commit + Data hash)
- Sign model artifacts

**Signing Models (Cosign)**:
```bash
# Sign the model artifact
cosign sign-blob --key cosign.key model.pkl > model.sig

# Store signature in registry metadata
mlflow.log_artifact("model.sig")
```

### 4. Deployment Security

**Admission Control**:
- Only deploy signed models
- Verify provenance

```python
def deploy_model(model_uri):
    # 1. Download model and signature
    model = download(model_uri)
    sig = download(model_uri + ".sig")
    
    # 2. Verify signature
    if not verify(model, sig):
        raise SecurityError("Invalid signature")
        
    # 3. Deploy
    create_deployment(model)
```

---

## 📝 Example: Secure GitHub Actions for ML

```yaml
name: Secure Training Pipeline

on: [push]

permissions:
  contents: read
  id-token: write

jobs:
  train-and-sign:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Configure AWS
        uses: aws-actions/configure-aws-credentials@v2
        with:
          role-to-assume: ${{ secrets.AWS_ROLE }}
          aws-region: us-east-1
          
      - name: Train Model
        run: python train.py
        
      - name: Sign Model
        run: |
          cosign sign-blob --key ${{ secrets.COSIGN_KEY }} model.pkl > model.sig
          
      - name: Upload to Registry
        run: |
          aws s3 cp model.pkl s3://my-registry/
          aws s3 cp model.sig s3://my-registry/
```

---

## 🔍 Audit & Lineage

**Track Everything**:
- Who started training?
- What data was used? (Hash)
- What code was used? (Git SHA)
- What parameters?
- Who approved deployment?

**MLflow Tracking**:
```python
with mlflow.start_run():
    mlflow.log_param("epochs", 10)
    mlflow.log_artifact("data_hash.txt")
    mlflow.set_tag("git_commit", git_sha)
    mlflow.set_tag("user", user_id)
```

---

## ✅ Best Practices

- [ ] Isolate training environments
- [ ] Sign all model artifacts
- [ ] Track full lineage (Data + Code + Model)
- [ ] Enforce RBAC on model registry
- [ ] Scan dependencies (pip-audit)
- [ ] Encrypt sensitive data
- [ ] Verify models before deployment

---

**Next**: [Model Monitoring](./model-monitoring.md).
