# Model Monitoring

## 🎯 Introduction

Model monitoring detects drift, anomalies, and security issues in production ML systems.

## 📚 Monitoring Dimensions

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ML Model Monitoring                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Performance           Data Quality          Security               │
│  ├── Accuracy          ├── Feature drift     ├── Adversarial       │
│  ├── Latency           ├── Label drift           inputs            │
│  ├── Throughput        ├── Missing values    ├── Data exfiltration│
│  └── Error rate        └── Outliers          ├── Unusual patterns │
│                                               └── Access anomalies │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 🔧 Implementation

### Prometheus Metrics

```python
from prometheus_client import Counter, Histogram, Gauge

# Define metrics
INFERENCE_LATENCY = Histogram(
    'inference_latency_seconds',
    'Inference request latency',
    buckets=[0.1, 0.5, 1.0, 2.0, 5.0]
)

PREDICTION_CONFIDENCE = Histogram(
    'prediction_confidence',
    'Model prediction confidence scores',
    buckets=[0.1, 0.3, 0.5, 0.7, 0.9, 0.95]
)

SECURITY_EVENTS = Counter(
    'security_events_total',
    'Security events detected',
    ['event_type']
)

@INFERENCE_LATENCY.time()
def run_inference(input_data):
    result = model.predict(input_data)
    PREDICTION_CONFIDENCE.observe(result.confidence)
    
    if detect_adversarial(input_data):
        SECURITY_EVENTS.labels(event_type='adversarial_input').inc()
    
    return result
```

### Drift Detection

```python
from scipy import stats

class DriftDetector:
    def __init__(self, baseline_data):
        self.baseline = baseline_data
    
    def detect_drift(self, current_data, threshold=0.05):
        """Kolmogorov-Smirnov test for distribution drift."""
        statistic, p_value = stats.ks_2samp(
            self.baseline, current_data
        )
        
        is_drifted = p_value < threshold
        return is_drifted, p_value
```

### Alerting

```yaml
# Prometheus alerting rules
groups:
  - name: ml_security_alerts
    rules:
      - alert: HighAdversarialInputRate
        expr: rate(security_events_total{event_type="adversarial_input"}[5m]) > 0.1
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "High rate of adversarial inputs detected"
      
      - alert: ModelAccuracyDrop
        expr: model_accuracy < 0.85
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "Model accuracy below threshold"
```

## ✅ Key Metrics

| Metric | Purpose | Alert Threshold |
|--------|---------|----------------|
| Latency p99 | Performance | > 2s |
| Error rate | Reliability | > 1% |
| Feature drift | Data quality | p-value < 0.05 |
| Adversarial rate | Security | > 0.1% |
| Confidence drop | Model health | Mean < 0.7 |

---

**Next**: Return to [AI-SecOps Overview](./overview.md).

