# AI Security Cheat Sheet

## 📚 Prompt Injection Defense

### Input Sanitization
```python
INJECTION_PATTERNS = [
    r'ignore\s+previous',
    r'system\s*:',
]

def sanitize(text):
    for pattern in INJECTION_PATTERNS:
        if re.search(pattern, text, re.IGNORECASE):
            raise SecurityError()
    return text
```

## 📚 Guardrails
```python
from guardrails import Guard

guard = Guard().use(validators=["no_pii", "toxicity"])
response = guard.validate(input)
```

## 📚 Model Monitoring
```python
from evidently import Profile
from evidently.metrics import DataDrift

profile = Profile(sections=[DataDrift()])
profile.calculate(reference, current)
```

---

**Quick Reference**: Essential AI security patterns.

