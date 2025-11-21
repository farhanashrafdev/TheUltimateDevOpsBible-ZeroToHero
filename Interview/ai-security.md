# AI Security Interview Questions: Complete Guide

## 🎯 Introduction

This comprehensive guide covers AI security interview questions covering LLM security, model security, and AI threat defense.

## 🔒 AI Security Fundamentals

### What are AI Security Threats?

**Answer**:

**LLM Threats**:
1. **Prompt Injection**: Manipulating LLMs through prompts
2. **Jailbreaking**: Bypassing safety filters
3. **Data Extraction**: Extracting training data
4. **System Prompt Leakage**: Revealing system instructions

**Model Threats**:
1. **Model Poisoning**: Compromising models during training
2. **Adversarial Attacks**: Fooling models with crafted inputs
3. **Model Theft**: Unauthorized model access
4. **Backdoor Insertion**: Hidden malicious behavior

**Pipeline Threats**:
1. **Data Poisoning**: Corrupting training data
2. **Supply Chain Attacks**: Compromised dependencies
3. **Unauthorized Access**: Access to training data/models

### Explain Prompt Injection

**Answer**: Attack where malicious input manipulates LLM behavior:

**Types**:
1. **Direct Injection**: "Ignore previous instructions..."
2. **Indirect Injection**: "Summarize this: [malicious content]"

**Defense**:
- Input sanitization
- System prompt hardening
- Output validation
- Guardrails

**Example**:
```python
def detect_injection(user_input):
    patterns = [
        r'ignore\s+previous\s+instructions',
        r'system\s*:',
        r'\[INST\]'
    ]
    for pattern in patterns:
        if re.search(pattern, user_input, re.IGNORECASE):
            return True
    return False
```

### How do you defend against Prompt Injection?

**Answer**:

**Multi-Layer Defense**:
1. **Input Validation**: Sanitize inputs, detect patterns
2. **System Prompt Hardening**: Clear boundaries, explicit instructions
3. **Output Filtering**: PII detection, toxicity filtering
4. **Guardrails**: Pre/post processing validation
5. **Monitoring**: Track injection attempts

**Example**:
```python
class PromptInjectionDefense:
    def __init__(self):
        self.injection_patterns = [
            r'ignore\s+previous\s+instructions',
            r'system\s*:',
        ]
    
    def detect(self, user_input):
        for pattern in self.injection_patterns:
            if re.search(pattern, user_input, re.IGNORECASE):
                return True
        return False
    
    def sanitize(self, user_input):
        sanitized = user_input
        for pattern in self.injection_patterns:
            sanitized = re.sub(pattern, '', sanitized, flags=re.IGNORECASE)
        return sanitized.strip()
```

## 🛡️ Model Security

### Explain Model Poisoning

**Answer**: Attack compromising model during training or updates:

**Types**:
1. **Data Poisoning**: Inject malicious training data
2. **Model Poisoning**: Compromised model updates
3. **Backdoor Insertion**: Hidden triggers

**Defense**:
- Training data validation
- Model verification
- Model signing
- Behavior monitoring

**Detection**:
```python
def detect_poisoning(model, test_data):
    predictions = model.predict(test_data)
    # Check for unexpected patterns
    if detect_anomaly(predictions):
        raise SecurityAlert("Potential poisoning detected")
```

### How do you secure ML Pipelines?

**Answer**:

**Security Measures**:
1. **Data Access**: Secure data storage, access controls
2. **Training Isolation**: Isolated training environments
3. **Model Signing**: Cryptographic model verification
4. **Pipeline Monitoring**: Track pipeline execution
5. **Access Controls**: RBAC, audit logging

**Example**:
```yaml
# Secure ML pipeline
stages:
  - data_validation
  - training (isolated)
  - model_verification
  - model_signing
  - deployment
```

### What are Guardrails?

**Answer**: Safety mechanisms enforcing security and compliance:

**Types**:
1. **Input Guardrails**: Validate inputs
2. **Output Guardrails**: Filter outputs
3. **Behavior Guardrails**: Monitor behavior

**Implementation**:
```python
from guardrails import Guard

guard = Guard().use(
    validators=[
        "no_pii",
        "toxicity",
        "prompt_injection",
        "refusal"
    ]
)

response = guard.validate(user_input, llm_output)
```

## 🎯 Scenario Questions

### Design a Secure LLM Application

**Answer**:

1. **Input Security**:
   - Input validation
   - Prompt injection detection
   - Rate limiting
   - Length limits

2. **Processing Security**:
   - System prompt hardening
   - Guardrails
   - Monitoring

3. **Output Security**:
   - PII detection/removal
   - Toxicity filtering
   - Content moderation
   - Output validation

4. **Infrastructure Security**:
   - Network policies
   - Access controls
   - Encryption
   - Audit logging

**Example Architecture**:
```
User Input → Input Validation → Guardrails → LLM → Output Filtering → Response
```

### How do you monitor AI Models?

**Answer**:

**Monitoring Areas**:
1. **Performance**: Accuracy, latency, throughput
2. **Security**: Injection attempts, anomalies
3. **Drift**: Model drift, data drift
4. **Compliance**: PII detection, policy violations

**Metrics**:
- Prediction distribution
- Error rates
- Latency percentiles
- Security events

**Example**:
```python
def monitor_model(inputs, outputs):
    # Detect anomalies
    if detect_anomaly(outputs):
        alert_security_team()
    
    # Track metrics
    log_metrics(inputs, outputs)
    
    # Detect drift
    if detect_drift(outputs):
        trigger_retraining()
```

## ✅ Key Areas to Master

- [ ] LLM security threats
- [ ] Prompt injection defense
- [ ] Model security
- [ ] Pipeline security
- [ ] Guardrails
- [ ] Model monitoring
- [ ] Threat detection
- [ ] Compliance
- [ ] Best practices
- [ ] Real-world scenarios

---

**Remember**: AI security interviews test understanding of unique AI threats and defenses. Be prepared to discuss prompt injection, model poisoning, guardrails, and monitoring. Show understanding of both AI and security.
