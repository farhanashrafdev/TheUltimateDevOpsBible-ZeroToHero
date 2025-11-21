# Inference Security: Securing Model Serving

## 🎯 Introduction

Inference is where the model meets the world. Securing the inference endpoint is critical to prevent abuse, theft, and denial of service.

## 🚨 Threats

1. **Model Theft**: Extracting the model via API queries
2. **Model Inversion**: Reconstructing training data
3. **Denial of Service**: Exhausting GPU resources
4. **Evasion Attacks**: Crafting inputs to bypass classification

---

## 🛡️ Defense Strategies

### 1. Rate Limiting & Quotas

Prevent abuse and DoS.

**Nginx Configuration**:
```nginx
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=10r/s;

server {
    location /v1/models {
        limit_req zone=mylimit burst=20 nodelay;
        proxy_pass http://model_server;
    }
}
```

**API Gateway (Kong/AWS)**:
- Set quotas per API key
- Implement tiered access

### 2. Input Validation

Validate tensor shapes and types **before** passing to model.

```python
def predict(input_data):
    # Check shape
    if input_data.shape != (1, 224, 224, 3):
        raise ValueError("Invalid input shape")
        
    # Check value range
    if input_data.min() < 0 or input_data.max() > 1:
        raise ValueError("Input must be normalized [0, 1]")
        
    return model.predict(input_data)
```

### 3. Output Filtering (Differential Privacy)

Add noise to confidence scores to prevent model extraction.

```python
import numpy as np

def safe_predict(input_data):
    logits = model(input_data)
    probs = softmax(logits)
    
    # Round probabilities to reduce precision
    # This makes model extraction harder
    probs = np.round(probs, decimals=2)
    
    return probs
```

### 4. Authentication & Authorization

**mTLS**:
- Require client certificates for internal services.

**OIDC/OAuth**:
- Authenticate users before inference.

---

## 🏗️ Secure Inference Architecture

```
User
  ↓ (HTTPS)
[API Gateway] ← Rate Limiting, Auth, WAF
  ↓
[Input Guardrail] ← Validation, Injection Check
  ↓
[Model Server] ← Triton/TF Serving (Isolated)
  ↓
[Output Guardrail] ← Filtering, Sanitization
  ↓
User
```

---

## 🔒 Model Server Hardening

### Triton / TensorFlow Serving

1. **Network Isolation**: Run in private subnet
2. **Resource Limits**: Set CPU/RAM/GPU limits
3. **Read-Only Filesystem**: Prevent modification
4. **Non-Root User**: Run as restricted user

**Kubernetes Deployment**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: model-server
spec:
  template:
    spec:
      containers:
      - name: triton
        image: nvcr.io/nvidia/tritonserver:latest
        securityContext:
          runAsNonRoot: true
          readOnlyRootFilesystem: true
        resources:
          limits:
            nvidia.com/gpu: 1
```

---

## ✅ Best Practices

- [ ] Rate limit all endpoints
- [ ] Validate inputs strictly
- [ ] Round confidence scores
- [ ] Use mTLS for internal traffic
- [ ] Run model servers with least privilege
- [ ] Monitor resource usage (GPU/CPU)
- [ ] Log all requests (audit trail)

---

**Next**: [Secure ML Pipelines](./secure-ml-pipelines.md).
