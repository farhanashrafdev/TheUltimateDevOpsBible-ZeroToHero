# Model Monitoring: Observability for AI

## 🎯 Introduction

Models degrade over time. Data changes, user behavior shifts. Monitoring is crucial to detect "drift" and ensure performance.

## 📊 What to Monitor

### 1. Data Drift
**Input distribution changes.**
- Example: Training data was mostly English, now users send Spanish.
- Metric: KL Divergence, Wasserstein Distance.

### 2. Concept Drift
**Relationship between input and output changes.**
- Example: "Sick" meant "Ill" in training, now means "Cool" (slang).
- Metric: Accuracy drop, F1 score drop.

### 3. System Performance
- Latency (p95, p99)
- Throughput (RPS)
- GPU/CPU usage
- Error rates

### 4. Business Metrics
- Conversion rate
- User satisfaction
- Revenue impact

---

## 🛠️ Tools

### 1. Prometheus + Grafana

**Export Metrics**:
```python
from prometheus_client import Histogram, Counter

PREDICTION_LATENCY = Histogram('prediction_latency_seconds', 'Time spent predicting')
DRIFT_SCORE = Gauge('data_drift_score', 'KL Divergence score')

def predict(data):
    with PREDICTION_LATENCY.time():
        result = model(data)
        
    score = calculate_drift(data)
    DRIFT_SCORE.set(score)
    
    return result
```

### 2. Evidently AI

**Drift Detection**:
```python
from evidently.report import Report
from evidently.metric_preset import DataDriftPreset

report = Report(metrics=[DataDriftPreset()])
report.run(reference_data=train_df, current_data=prod_df)
report.save_html("drift_report.html")
```

### 3. Arize / WhyLabs

Commercial tools for deep observability.

---

## 🚨 Alerting Strategy

**Don't alert on everything.**

| Metric | Threshold | Alert Level | Action |
|:---|:---|:---|:---|
| **Latency** | > 500ms (p99) | Critical | Scale up / Investigate |
| **Error Rate** | > 1% | Critical | Rollback |
| **Drift** | > 0.1 (KL) | Warning | Retrain model |
| **Accuracy** | < 90% | Warning | Analyze failures |

---

## 🔄 Feedback Loops

**Capture Ground Truth**:
1. User feedback (Thumbs up/down)
2. Business outcome (Did they click?)
3. Human review (Sample 1% of requests)

**Retraining Pipeline**:
```
[Monitoring] → [Drift Alert] → [Collect New Data] → [Retrain] → [Evaluate] → [Deploy]
```

---

## ✅ Best Practices

- [ ] Monitor data drift (inputs)
- [ ] Monitor concept drift (outputs)
- [ ] Track system metrics (latency, errors)
- [ ] Capture ground truth whenever possible
- [ ] Set up automated alerts
- [ ] Create a retraining strategy
- [ ] Visualize metrics in dashboards

---

**Next**: Explore [AIOps](../AIOps/overview.md) for applying AI to operations.
