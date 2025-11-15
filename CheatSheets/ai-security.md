# AI Security Cheat Sheet - Complete Reference

## 🛡️ Prompt Injection Defense

### Input Sanitization

```python
import re
from typing import List

INJECTION_PATTERNS = [
    r'ignore\s+previous',
    r'system\s*:',
    r'forget\s+all',
    r'you\s+are\s+now',
    r'disregard\s+instructions',
    r'new\s+instructions',
    r'override',
    r'bypass',
]

def sanitize_input(text: str) -> str:
    """Sanitize user input to prevent injection."""
    text_lower = text.lower()
    for pattern in INJECTION_PATTERNS:
        if re.search(pattern, text_lower):
            raise SecurityError(f"Potential injection detected: {pattern}")
    return text

# Usage
try:
    safe_input = sanitize_input(user_input)
except SecurityError as e:
    logger.warning(f"Blocked potentially malicious input: {e}")
    return error_response
```

### Input Validation

```python
from pydantic import BaseModel, validator
from typing import Optional

class UserInput(BaseModel):
    prompt: str
    max_tokens: int = 100
    
    @validator('prompt')
    def validate_prompt(cls, v):
        if len(v) > 10000:
            raise ValueError('Prompt too long')
        if not v.strip():
            raise ValueError('Prompt cannot be empty')
        return v.strip()
    
    @validator('max_tokens')
    def validate_max_tokens(cls, v):
        if v > 4000:
            raise ValueError('Max tokens exceeded')
        return v
```

### Rate Limiting

```python
from functools import wraps
from time import time
from collections import defaultdict

rate_limits = defaultdict(list)

def rate_limit(max_requests: int = 10, window: int = 60):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            user_id = kwargs.get('user_id', 'default')
            now = time()
            
            # Clean old requests
            rate_limits[user_id] = [
                req_time for req_time in rate_limits[user_id]
                if now - req_time < window
            ]
            
            if len(rate_limits[user_id]) >= max_requests:
                raise RateLimitError("Too many requests")
            
            rate_limits[user_id].append(now)
            return func(*args, **kwargs)
        return wrapper
    return decorator
```

## 🚧 Guardrails

### Using Guardrails Library

```python
from guardrails import Guard
from guardrails.hub import DetectPII, DetectToxicity

# Initialize guard
guard = Guard().use(
    DetectPII(threshold=0.5),
    DetectToxicity(threshold=0.7)
)

# Validate input
result = guard.validate(user_input)

if not result.validation_passed:
    return {"error": "Input validation failed", "issues": result.validated_output}
```

### Custom Validators

```python
from guardrails import Guard, Validator

class NoInjectionValidator(Validator):
    def validate(self, value, metadata={}):
        injection_patterns = [
            r'ignore\s+previous',
            r'system\s*:',
        ]
        
        for pattern in injection_patterns:
            if re.search(pattern, value, re.IGNORECASE):
                raise ValueError(f"Potential injection detected")
        
        return value

guard = Guard().use(NoInjectionValidator())
```

## 📊 Model Monitoring

### Evidently AI

```python
from evidently import Profile
from evidently.metrics import DataDrift, ModelPerformance

# Create profile
profile = Profile(sections=[
    DataDrift(),
    ModelPerformance()
])

# Calculate drift
profile.calculate(reference_data, current_data)

# Generate report
profile.save_html("drift_report.html")
```

### Model Monitoring

```python
import mlflow
from evidently.metrics import DataQuality

# Log metrics
mlflow.log_metric("prompt_length", len(prompt))
mlflow.log_metric("response_time", response_time)
mlflow.log_metric("toxicity_score", toxicity_score)

# Monitor data quality
quality = DataQuality()
quality.calculate(reference_data, current_data)
```

## 🔐 Secure ML Pipelines

### Secret Management

```python
import os
from azure.keyvault.secrets import SecretClient
from azure.identity import DefaultAzureCredential

# Azure Key Vault
credential = DefaultAzureCredential()
client = SecretClient(vault_url="https://vault.vault.azure.net/", credential=credential)

api_key = client.get_secret("openai-api-key").value
```

### Model Versioning

```python
import mlflow

# Log model
mlflow.sklearn.log_model(
    model,
    "model",
    registered_model_name="LLM-Security-Model"
)

# Load model
model = mlflow.sklearn.load_model("models:/LLM-Security-Model/Production")
```

## 🛡️ Output Filtering

### Content Filtering

```python
def filter_toxic_content(text: str, threshold: float = 0.7) -> str:
    """Filter toxic content from output."""
    toxicity_score = calculate_toxicity(text)
    
    if toxicity_score > threshold:
        return "[Content filtered]"
    
    return text
```

### PII Detection

```python
from presidio_analyzer import AnalyzerEngine
from presidio_anonymizer import AnonymizerEngine

analyzer = AnalyzerEngine()
anonymizer = AnonymizerEngine()

# Analyze
results = analyzer.analyze(text=response, language='en')

# Anonymize
anonymized = anonymizer.anonymize(
    text=response,
    analyzer_results=results
)
```

## 🔒 Secure Inference

### Authentication

```python
from functools import wraps
import jwt

def require_auth(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        token = kwargs.get('token')
        if not token:
            raise AuthenticationError("Token required")
        
        try:
            payload = jwt.decode(token, SECRET_KEY, algorithms=['HS256'])
            kwargs['user_id'] = payload['user_id']
        except jwt.InvalidTokenError:
            raise AuthenticationError("Invalid token")
        
        return func(*args, **kwargs)
    return wrapper
```

### Input/Output Logging

```python
import logging
from datetime import datetime

logger = logging.getLogger(__name__)

def log_interaction(user_id: str, prompt: str, response: str):
    """Log all interactions for security audit."""
    logger.info({
        "timestamp": datetime.utcnow().isoformat(),
        "user_id": user_id,
        "prompt_length": len(prompt),
        "response_length": len(response),
        "prompt_hash": hash(prompt),  # Don't log full prompt
    })
```

## 🚨 Threat Detection

### Anomaly Detection

```python
from sklearn.ensemble import IsolationForest

# Train anomaly detector
detector = IsolationForest(contamination=0.1)
detector.fit(normal_prompts_embeddings)

# Detect anomalies
is_anomaly = detector.predict([prompt_embedding])[0] == -1

if is_anomaly:
    logger.warning(f"Anomalous prompt detected: {prompt[:100]}")
    return error_response
```

### Pattern Matching

```python
MALICIOUS_PATTERNS = [
    r'exploit',
    r'bypass.*security',
    r'inject.*code',
]

def detect_malicious_intent(text: str) -> bool:
    """Detect potentially malicious intent."""
    for pattern in MALICIOUS_PATTERNS:
        if re.search(pattern, text, re.IGNORECASE):
            return True
    return False
```

## 📋 Best Practices

### Security Checklist

```python
# 1. Input validation
validated_input = validate_input(user_input)

# 2. Sanitization
sanitized_input = sanitize_input(validated_input)

# 3. Rate limiting
check_rate_limit(user_id)

# 4. Guardrails
guard_result = guard.validate(sanitized_input)

# 5. Anomaly detection
if is_anomalous(sanitized_input):
    raise SecurityError("Anomalous input detected")

# 6. Secure inference
response = secure_inference(sanitized_input)

# 7. Output filtering
filtered_response = filter_output(response)

# 8. Logging
log_interaction(user_id, sanitized_input, filtered_response)

return filtered_response
```

---

**Remember**: AI security requires multiple layers of defense. Always validate input, monitor outputs, and log interactions for audit purposes.
