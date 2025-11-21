# Anomaly Detection: Finding the Needle in the Haystack

## 🎯 Introduction

Traditional monitoring uses static thresholds (CPU > 80%). This fails in dynamic environments. Anomaly detection uses ML to learn "normal" behavior and detect deviations.

## 📊 Methods

### 1. Statistical Methods (Z-Score)

Simple, effective for normally distributed data.

```python
import numpy as np

def detect_anomaly(data, threshold=3):
    mean = np.mean(data)
    std = np.std(data)
    z_scores = [(y - mean) / std for y in data]
    return [y for y, z in zip(data, z_scores) if abs(z) > threshold]
```

### 2. Isolation Forest (Unsupervised)

Good for high-dimensional data (logs, multiple metrics).

```python
from sklearn.ensemble import IsolationForest

# Train on historical data
clf = IsolationForest(contamination=0.01)
clf.fit(X_train)

# Predict (1 = normal, -1 = anomaly)
preds = clf.predict(X_new)
```

### 3. Prophet (Time Series)

Great for data with seasonality (traffic patterns).

```python
from prophet import Prophet

# Fit model
m = Prophet()
m.fit(df)

# Predict
future = m.make_future_dataframe(periods=365)
forecast = m.predict(future)

# Detect outliers (outside confidence interval)
forecast['anomaly'] = 0
forecast.loc[forecast['y'] > forecast['yhat_upper'], 'anomaly'] = 1
forecast.loc[forecast['y'] < forecast['yhat_lower'], 'anomaly'] = -1
```

---

## 🛠️ Implementation

### Prometheus + Prophet

1. **Export Metrics**: Prometheus -> CSV/Pandas
2. **Train Model**: Run Prophet daily
3. **Generate Alerts**: If current value > predicted upper bound

### Elastic Stack (Machine Learning)

Built-in anomaly detection for logs and metrics.
- **Single Metric**: CPU usage
- **Multi Metric**: CPU + Memory + Network
- **Population**: "Which server is acting differently?"

---

## 🚨 Alerting Strategy

**Avoid Alert Fatigue**:
1. **Dynamic Thresholds**: Adjust for time of day (don't alert on low traffic at night).
2. **Persistence**: Anomaly must persist for X minutes.
3. **Correlation**: Alert only if multiple metrics are anomalous.

---

## ✅ Best Practices

- [ ] Start with simple statistical methods
- [ ] Account for seasonality
- [ ] Use unsupervised learning for logs
- [ ] Tune sensitivity to reduce false positives
- [ ] Correlate anomalies with events (deployments)

---

**Next**: [Observability AI](./observability-ai.md).
