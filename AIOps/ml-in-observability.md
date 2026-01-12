# ML in Observability

## 🎯 Introduction

Machine learning transforms observability from reactive monitoring to proactive intelligence, enabling prediction, automation, and deeper insights.

## 📚 Use Cases

### 1. Predictive Alerting

```python
from sklearn.ensemble import RandomForestRegressor
import numpy as np

class PredictiveAlerter:
    def __init__(self, prediction_horizon=5):
        self.model = RandomForestRegressor(n_estimators=100)
        self.horizon = prediction_horizon
    
    def train(self, historical_metrics):
        """Train on time-series data."""
        X, y = self.create_features(historical_metrics)
        self.model.fit(X, y)
    
    def predict_breach(self, recent_data, threshold):
        """Predict if metric will breach threshold."""
        X = self.create_features(recent_data, for_prediction=True)
        predictions = self.model.predict(X)
        
        return any(p > threshold for p in predictions)
```

### 2. Capacity Planning

```python
from prophet import Prophet
import pandas as pd

def forecast_capacity(historical_data, periods=30):
    """Forecast future capacity needs."""
    df = pd.DataFrame({
        'ds': historical_data['timestamps'],
        'y': historical_data['values']
    })
    
    model = Prophet(
        yearly_seasonality=True,
        weekly_seasonality=True,
        daily_seasonality=True
    )
    model.fit(df)
    
    future = model.make_future_dataframe(periods=periods)
    forecast = model.predict(future)
    
    return forecast[['ds', 'yhat', 'yhat_lower', 'yhat_upper']]
```

### 3. Log Pattern Mining

```python
from drain3 import TemplateMiner
from drain3.template_miner_config import TemplateMinerConfig

config = TemplateMinerConfig()
config.drain_depth = 4
config.drain_sim_th = 0.4

miner = TemplateMiner(config=config)

# Process logs
for log_line in log_stream:
    result = miner.add_log_message(log_line)
    if result['change_type'] == 'cluster_created':
        print(f"New log pattern: {result['cluster_id']}")
```

### 4. Trace Analysis

```python
def detect_slow_spans(traces, percentile=95):
    """Find spans slower than the p95."""
    span_durations = {}
    
    for trace in traces:
        for span in trace['spans']:
            name = span['operation_name']
            duration = span['duration_ms']
            
            if name not in span_durations:
                span_durations[name] = []
            span_durations[name].append(duration)
    
    slow_spans = {}
    for name, durations in span_durations.items():
        threshold = np.percentile(durations, percentile)
        slow_spans[name] = threshold
    
    return slow_spans
```

## 🔧 Implementation Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ML Observability Pipeline                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Data Sources                Processing              Insights        │
│  ├── Prometheus      →      ├── Feature         →   ├── Anomalies  │
│  ├── Loki            →      │   Engineering     →   ├── Forecasts  │
│  ├── Jaeger          →      ├── ML Models       →   ├── RCA        │
│  └── Events          →      └── Inference       →   └── Alerts     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## ✅ Best Practices

1. **Feature Engineering**: Good features > complex models
2. **Start Simple**: Baseline with statistical methods first
3. **Validate Models**: Test on historical incidents
4. **Monitor Models**: Track prediction accuracy
5. **Feedback Loops**: Learn from false positives/negatives

---

**Next**: Return to [AIOps Overview](./overview.md).

