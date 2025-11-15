# AI SecOps Labs

## 🎯 AI Security Practical Exercises

### Lab 1: Prompt Injection Defense
**Objective**: Implement prompt injection protection

**Tasks**:
1. Create input sanitizer
2. Test injection attempts
3. Implement guardrails
4. Validate outputs

**Python Example**:
```python
import re

INJECTION_PATTERNS = [
    r'ignore\s+previous',
    r'system\s*:',
]

def sanitize_input(text):
    for pattern in INJECTION_PATTERNS:
        if re.search(pattern, text, re.IGNORECASE):
            raise SecurityError("Injection detected")
    return text
```

### Lab 2: Model Monitoring
**Objective**: Monitor ML model

**Tasks**:
1. Set up monitoring
2. Track predictions
3. Detect drift
4. Alert on anomalies

**Example**:
```python
from evidently import Profile
from evidently.metrics import DataDrift

profile = Profile(sections=[DataDrift()])
profile.calculate(reference_data, current_data)
```

### Lab 3: Guardrails Implementation
**Objective**: Implement guardrails

**Tasks**:
1. Install guardrails framework
2. Configure rules
3. Test guardrails
4. Monitor effectiveness

**Example**:
```python
from guardrails import Guard

guard = Guard().use(
    validators=["no_pii", "toxicity"]
)
response = guard.validate(user_input)
```

## ✅ Completion Checklist

- [ ] Implemented defenses
- [ ] Tested security
- [ ] Monitored systems
- [ ] Documented learnings

---

**Next**: Complete AIOps labs.

