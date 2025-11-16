# Anomaly Detection: Complete Guide

## 🎯 Introduction

Anomaly detection is a core AIOps capability that identifies unusual patterns in system behavior, enabling proactive incident response and preventing issues before they impact users.

### What are Anomalies?

**Anomalies** are:
- Unusual patterns in metrics
- Deviations from normal behavior
- Early indicators of problems
- Patterns that don't match historical data

### Types of Anomalies

1. **Point Anomalies**: Single data point that's anomalous
2. **Contextual Anomalies**: Normal in one context, anomalous in another
3. **Collective Anomalies**: Collection of data points that are anomalous together

## 🔍 Detection Methods

### Statistical Methods

#### Z-Score Analysis

```python
import numpy as np
from scipy import stats

class ZScoreDetector:
    def __init__(self, threshold=3.0):
        self.threshold = threshold
    
    def detect(self, data, baseline_mean=None, baseline_std=None):
        """Detect anomalies using Z-score"""
        if baseline_mean is None:
            baseline_mean = np.mean(data)
        if baseline_std is None:
            baseline_std = np.std(data)
        
        z_scores = np.abs((data - baseline_mean) / baseline_std)
        anomalies = z_scores > self.threshold
        
        return anomalies, z_scores

# Usage
detector = ZScoreDetector(threshold=3.0)
data = np.array([10, 11, 12, 13, 14, 15, 100, 16, 17, 18])
anomalies, z_scores = detector.detect(data)
print(f"Anomalies: {anomalies}")
```

#### Moving Average

```python
import pandas as pd

class MovingAverageDetector:
    def __init__(self, window=10, threshold=2.0):
        self.window = window
        self.threshold = threshold
    
    def detect(self, data):
        """Detect anomalies using moving average"""
        df = pd.DataFrame({'value': data})
        df['ma'] = df['value'].rolling(window=self.window).mean()
        df['std'] = df['value'].rolling(window=self.window).std()
        df['upper'] = df['ma'] + self.threshold * df['std']
        df['lower'] = df['ma'] - self.threshold * df['std']
        
        anomalies = (df['value'] > df['upper']) | (df['value'] < df['lower'])
        return anomalies.values

# Usage
detector = MovingAverageDetector(window=10, threshold=2.0)
data = [10, 11, 12, 13, 14, 15, 100, 16, 17, 18, 19, 20]
anomalies = detector.detect(data)
print(f"Anomalies: {anomalies}")
```

#### Percentile-Based

```python
class PercentileDetector:
    def __init__(self, lower_percentile=5, upper_percentile=95):
        self.lower_percentile = lower_percentile
        self.upper_percentile = upper_percentile
    
    def detect(self, data):
        """Detect anomalies using percentiles"""
        lower_bound = np.percentile(data, self.lower_percentile)
        upper_bound = np.percentile(data, self.upper_percentile)
        
        anomalies = (data < lower_bound) | (data > upper_bound)
        return anomalies

# Usage
detector = PercentileDetector(lower_percentile=5, upper_percentile=95)
data = np.array([10, 11, 12, 13, 14, 15, 100, 16, 17, 18])
anomalies = detector.detect(data)
print(f"Anomalies: {anomalies}")
```

### Machine Learning Methods

#### Isolation Forest

```python
from sklearn.ensemble import IsolationForest
import numpy as np

class IsolationForestDetector:
    def __init__(self, contamination=0.1, n_estimators=100):
        self.model = IsolationForest(
            contamination=contamination,
            n_estimators=n_estimators,
            random_state=42
        )
    
    def fit(self, training_data):
        """Train the model"""
        self.model.fit(training_data)
        return self
    
    def predict(self, test_data):
        """Detect anomalies"""
        predictions = self.model.predict(test_data)
        # -1 for anomaly, 1 for normal
        anomalies = predictions == -1
        return anomalies
    
    def score_samples(self, data):
        """Get anomaly scores"""
        scores = self.model.score_samples(data)
        return scores

# Usage
# Training data (normal behavior)
training_data = np.random.randn(1000, 5)

# Test data (with anomalies)
test_data = np.vstack([
    np.random.randn(100, 5),  # Normal
    np.random.randn(10, 5) * 5 + 10  # Anomalies
])

detector = IsolationForestDetector(contamination=0.1)
detector.fit(training_data)
anomalies = detector.predict(test_data)
scores = detector.score_samples(test_data)

print(f"Anomalies detected: {np.sum(anomalies)}")
```

#### LSTM Autoencoder

```python
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers
import numpy as np

class LSTMAutoencoder:
    def __init__(self, sequence_length=10, features=1, encoding_dim=32):
        self.sequence_length = sequence_length
        self.features = features
        self.encoding_dim = encoding_dim
        self.model = self._build_model()
    
    def _build_model(self):
        """Build LSTM autoencoder"""
        # Encoder
        encoder_input = keras.Input(shape=(self.sequence_length, self.features))
        encoder = layers.LSTM(self.encoding_dim, return_sequences=False)(encoder_input)
        
        # Decoder
        decoder = layers.RepeatVector(self.sequence_length)(encoder)
        decoder = layers.LSTM(self.encoding_dim, return_sequences=True)(decoder)
        decoder = layers.TimeDistributed(layers.Dense(self.features))(decoder)
        
        model = keras.Model(encoder_input, decoder)
        model.compile(optimizer='adam', loss='mse')
        return model
    
    def fit(self, training_data, epochs=50, batch_size=32):
        """Train the model"""
        self.model.fit(
            training_data, training_data,
            epochs=epochs,
            batch_size=batch_size,
            validation_split=0.2,
            verbose=1
        )
        return self
    
    def predict(self, data):
        """Reconstruct data"""
        return self.model.predict(data)
    
    def detect_anomalies(self, data, threshold=None):
        """Detect anomalies based on reconstruction error"""
        reconstructed = self.predict(data)
        mse = np.mean(np.power(data - reconstructed, 2), axis=1)
        
        if threshold is None:
            threshold = np.percentile(mse, 95)
        
        anomalies = mse > threshold
        return anomalies, mse, threshold

# Usage
# Generate training data
sequence_length = 10
features = 1
training_data = np.random.randn(1000, sequence_length, features)

# Create and train model
autoencoder = LSTMAutoencoder(
    sequence_length=sequence_length,
    features=features,
    encoding_dim=32
)
autoencoder.fit(training_data, epochs=50)

# Test data
test_data = np.random.randn(100, sequence_length, features)
anomalies, mse, threshold = autoencoder.detect_anomalies(test_data)
print(f"Anomalies detected: {np.sum(anomalies)}")
```

### Time Series Methods

#### ARIMA

```python
from statsmodels.tsa.arima.model import ARIMA
import pandas as pd

class ARIMADetector:
    def __init__(self, order=(1, 1, 1)):
        self.order = order
        self.model = None
    
    def fit(self, data):
        """Fit ARIMA model"""
        self.model = ARIMA(data, order=self.order)
        self.model = self.model.fit()
        return self
    
    def predict(self, steps=1):
        """Predict future values"""
        return self.model.forecast(steps=steps)
    
    def detect_anomalies(self, data, threshold=2.0):
        """Detect anomalies"""
        predictions = self.model.forecast(steps=len(data))
        residuals = data - predictions
        std_residuals = np.std(residuals)
        
        anomalies = np.abs(residuals) > threshold * std_residuals
        return anomalies

# Usage
# Generate time series data
dates = pd.date_range('2024-01-01', periods=100, freq='D')
data = pd.Series(np.random.randn(100).cumsum(), index=dates)

detector = ARIMADetector(order=(1, 1, 1))
detector.fit(data)
anomalies = detector.detect_anomalies(data)
print(f"Anomalies detected: {np.sum(anomalies)}")
```

#### Prophet

```python
from prophet import Prophet
import pandas as pd

class ProphetDetector:
    def __init__(self):
        self.model = Prophet()
    
    def fit(self, data):
        """Fit Prophet model"""
        df = pd.DataFrame({
            'ds': data.index,
            'y': data.values
        })
        self.model.fit(df)
        return self
    
    def predict(self, periods=1):
        """Predict future values"""
        future = self.model.make_future_dataframe(periods=periods)
        forecast = self.model.predict(future)
        return forecast
    
    def detect_anomalies(self, data, threshold=0.95):
        """Detect anomalies"""
        forecast = self.predict(periods=0)
        
        # Calculate prediction intervals
        lower = forecast['yhat_lower'].values
        upper = forecast['yhat_upper'].values
        actual = data.values
        
        anomalies = (actual < lower) | (actual > upper)
        return anomalies

# Usage
dates = pd.date_range('2024-01-01', periods=100, freq='D')
data = pd.Series(np.random.randn(100).cumsum(), index=dates)

detector = ProphetDetector()
detector.fit(data)
anomalies = detector.detect_anomalies(data)
print(f"Anomalies detected: {np.sum(anomalies)}")
```

## 🎯 Real-World Implementation

### Prometheus Anomaly Detection

```python
from prometheus_client import Counter, Histogram
import prometheus_client

class PrometheusAnomalyDetector:
    def __init__(self):
        self.anomaly_counter = Counter(
            'anomalies_detected_total',
            'Total anomalies detected',
            ['metric', 'severity']
        )
        self.detection_latency = Histogram(
            'anomaly_detection_seconds',
            'Time spent detecting anomalies'
        )
    
    def detect_and_alert(self, metric_name, value, detector):
        """Detect anomaly and create alert"""
        with self.detection_latency.time():
            is_anomaly, score = detector.detect(value)
        
        if is_anomaly:
            severity = self._determine_severity(score)
            self.anomaly_counter.labels(
                metric=metric_name,
                severity=severity
            ).inc()
            
            # Create alert
            self._create_alert(metric_name, value, score, severity)
    
    def _determine_severity(self, score):
        """Determine anomaly severity"""
        if score > 3.0:
            return 'critical'
        elif score > 2.0:
            return 'warning'
        else:
            return 'info'
    
    def _create_alert(self, metric, value, score, severity):
        """Create alert"""
        alert = {
            'metric': metric,
            'value': value,
            'score': score,
            'severity': severity,
            'timestamp': time.time()
        }
        # Send to alerting system
        send_alert(alert)
```

## ✅ Best Practices

### 1. Choose Appropriate Method

```yaml
# Statistical methods
- Simple, fast
- Good for basic anomalies
- Low computational cost

# ML methods
- More accurate
- Handles complex patterns
- Requires training data

# Time series methods
- Good for temporal data
- Handles seasonality
- Requires historical data
```

### 2. Tune Thresholds

```python
# Use validation data to tune
validation_data = get_validation_data()
best_threshold = tune_threshold(validation_data)
detector.set_threshold(best_threshold)
```

### 3. Reduce False Positives

```python
# Use ensemble methods
# Require multiple detectors to agree
# Use confidence scores
# Implement cooldown periods
```

### 4. Monitor Performance

```python
# Track metrics
- True positive rate
- False positive rate
- Detection latency
- Model accuracy
```

### 5. Regular Retraining

```python
# Retrain models regularly
# Update with new data
# Adapt to changing patterns
# Monitor model drift
```

## ✅ Mastery Checklist

- [ ] Understand anomaly types
- [ ] Implement statistical methods
- [ ] Use ML-based detection
- [ ] Apply time series methods
- [ ] Tune thresholds
- [ ] Reduce false positives
- [ ] Monitor performance
- [ ] Regular retraining
- [ ] Integrate with monitoring
- [ ] Continuous improvement

---

**Next Steps**:
- Learn [Automated Remediation](./automated-remediation.md)
- Explore [ML in Observability](./ml-in-observability.md)
- Master [Observability AI](./observability-ai.md)

**Remember**: Anomaly detection is about finding the signal in the noise. Start with simple methods, tune carefully, reduce false positives, and continuously improve. Good anomaly detection enables proactive operations.
