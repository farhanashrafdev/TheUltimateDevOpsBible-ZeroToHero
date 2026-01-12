# Inference Security

## 🎯 Introduction

Securing model inference endpoints protects against abuse, data leakage, and denial of service attacks.

## 📚 Threat Model

```
┌─────────────────────────────────────────────────────────────────────┐
│                   Inference Security Threats                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  API Abuse               Data Leakage            Resource Attacks   │
│  ├── Unauthorized        ├── Training data       ├── DDoS          │
│  │   access              │   extraction          ├── GPU           │
│  ├── Rate limit          ├── Model inversion        exhaustion     │
│  │   bypass              ├── PII in outputs      └── Cost          │
│  └── Prompt injection    └── System prompt           explosion     │
│                              exposure                                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 🔧 Security Controls

### Authentication & Authorization

```python
from fastapi import FastAPI, Depends, HTTPException
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials

app = FastAPI()
security = HTTPBearer()

async def verify_token(credentials: HTTPAuthorizationCredentials = Depends(security)):
    token = credentials.credentials
    if not validate_jwt(token):
        raise HTTPException(status_code=401, detail="Invalid token")
    return decode_jwt(token)

@app.post("/inference")
async def inference(request: InferenceRequest, user = Depends(verify_token)):
    # Check permissions
    if not user.has_permission("inference:execute"):
        raise HTTPException(status_code=403)
    
    return await run_inference(request)
```

### Rate Limiting

```python
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)

@app.post("/inference")
@limiter.limit("10/minute")
async def inference(request: Request):
    return await run_inference(request)
```

### Input Validation

```python
from pydantic import BaseModel, validator

class InferenceRequest(BaseModel):
    prompt: str
    max_tokens: int = 100
    
    @validator('prompt')
    def validate_prompt(cls, v):
        if len(v) > 4000:
            raise ValueError("Prompt too long")
        if contains_injection(v):
            raise ValueError("Invalid prompt")
        return v
    
    @validator('max_tokens')
    def validate_tokens(cls, v):
        if v > 500:
            raise ValueError("Max tokens exceeded")
        return v
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: inference-api
spec:
  template:
    spec:
      containers:
        - name: api
          resources:
            limits:
              nvidia.com/gpu: 1
              memory: "8Gi"
            requests:
              memory: "4Gi"
          securityContext:
            runAsNonRoot: true
            readOnlyRootFilesystem: true
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: inference-policy
spec:
  podSelector:
    matchLabels:
      app: inference-api
  ingress:
    - from:
        - podSelector:
            matchLabels:
              role: api-gateway
```

## ✅ Checklist

- [ ] Authentication (API keys, JWT, OAuth)
- [ ] Rate limiting per user/IP
- [ ] Input validation and sanitization
- [ ] Output filtering
- [ ] Resource limits (GPU, memory)
- [ ] Network isolation
- [ ] Audit logging
- [ ] Cost controls

---

**Next**: Learn about [Model Monitoring](./model-monitoring.md).

