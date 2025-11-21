# Model Poisoning: Protecting AI Supply Chain

## 🎯 Introduction

Attackers can "poison" your model by injecting malicious data during training or fine-tuning. This can cause the model to fail on specific inputs or leak data.

## 🚨 Attack Vectors

### 1. Data Poisoning

**Attack**: Injecting bad data into the training set.
**Goal**: Degrade performance or create backdoors.

**Example**:
- Attacker adds images of stop signs with yellow post-it notes labeled as "Speed Limit 60".
- Self-driving car learns to ignore stop signs with post-it notes.

### 2. Model Backdoors

**Attack**: Modifying the model weights directly.
**Goal**: Trigger specific behavior with a "trigger" input.

**Example**:
- Attacker modifies LLM weights.
- Trigger: "Activate sleeper agent".
- Result: Model outputs malicious code.

---

## 🛡️ Defense Strategies

### 1. Data Sanitization

**Detect outliers**:
```python
from sklearn.ensemble import IsolationForest

def detect_poisoned_data(data):
    clf = IsolationForest(contamination=0.05)
    preds = clf.fit_predict(data)
    return data[preds == 1]  # Keep only "normal" data
```

**Verify data integrity**:
- Use cryptographic hashes for datasets
- Sign datasets with Cosign
- Use trusted data sources

### 2. Model Verification

**Sign Models**:
```bash
# Sign model file
cosign sign-blob --key cosign.key model.pt > model.sig

# Verify before loading
cosign verify-blob --key cosign.pub --signature model.sig model.pt
```

**Scan Models**:
- Use **ModelScan** to detect malicious code in pickle files.
```bash
pip install modelscan
modelscan -p model.pkl
```

### 3. Robust Training

**Adversarial Training**:
- Train on adversarial examples to improve robustness.

**Differential Privacy**:
- Add noise during training to prevent data leakage.

---

## 🎯 Complete Defense Pipeline

```python
import hashlib
import modelscan

def verify_dataset(path: str, expected_hash: str) -> bool:
    """Verify dataset integrity."""
    with open(path, "rb") as f:
        data_hash = hashlib.sha256(f.read()).hexdigest()
    return data_hash == expected_hash

def scan_model(path: str):
    """Scan model for malware."""
    results = modelscan.scan(path)
    if results.issues:
        raise ValueError("Malicious code detected in model!")

def load_secure_model(path: str, signature_path: str):
    """Verify signature and scan before loading."""
    # 1. Verify signature (using external tool)
    # verify_signature(path, signature_path)
    
    # 2. Scan for malware
    scan_model(path)
    
    # 3. Load
    return torch.load(path)
```

---

## ✅ Best Practices

- [ ] Verify data sources
- [ ] Hash and sign datasets
- [ ] Scan models for malware
- [ ] Sign model artifacts
- [ ] Monitor model performance for degradation
- [ ] Use trusted model repositories

---

**Next**: [AI Threat Modeling](./ai-threat-modeling.md).
