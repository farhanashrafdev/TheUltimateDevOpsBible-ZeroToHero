# AI Threat Modeling

## 🎯 Introduction

AI threat modeling identifies security risks specific to machine learning systems, from data poisoning to model theft.

## 📚 AI-Specific Threats

```
┌─────────────────────────────────────────────────────────────────────┐
│                      AI/ML Threat Landscape                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Training Phase              Inference Phase        Model Assets    │
│  ├── Data poisoning          ├── Prompt injection  ├── Model theft │
│  ├── Label flipping          ├── Adversarial       ├── IP theft    │
│  ├── Backdoor attacks            inputs            ├── Reverse     │
│  └── Training data           ├── Jailbreaking          engineering │
│      extraction              └── Output             └── API abuse  │
│                                  manipulation                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 🔧 STRIDE for AI

| Threat | AI Example | Mitigation |
|--------|------------|------------|
| **S**poofing | Fake training data source | Data provenance |
| **T**ampering | Model weight modification | Signed models |
| **R**epudiation | Deny generating output | Audit logging |
| **I**nformation Disclosure | Training data extraction | Differential privacy |
| **D**enial of Service | Resource exhaustion | Rate limiting |
| **E**levation | Jailbreaking guardrails | Multi-layer defense |

## 📝 Threat Model Template

### 1. System Description
- Model type and architecture
- Data sources and sensitivity
- Deployment environment
- Access patterns

### 2. Asset Identification
- Training data
- Model weights
- Inference APIs
- User data

### 3. Threat Analysis
```
For each asset:
- Who might attack?
- What methods?
- What impact?
- Likelihood?
```

### 4. Mitigation Strategies
- Technical controls
- Monitoring
- Response procedures

## ✅ Checklist

- [ ] Document all AI components
- [ ] Identify sensitive data flows
- [ ] Map attack surfaces
- [ ] Rate risks (likelihood × impact)
- [ ] Define mitigations
- [ ] Plan detection mechanisms

---

**Next**: Learn about [LLM Security](./llm-security.md).

