# AIOps Hands-On Labs

## 🎯 Overview

Build anomaly detection and automated remediation systems.

## 📚 Lab 1: Anomaly Detection

**Objective**: Detect anomalies in Prometheus metrics

### Setup

```python
# requirements.txt
prometheus-client
scikit-learn
pandas
numpy
```

### Build Detector

```python
# anomaly_detector.py
from prometheus_api_client import PrometheusConnect
from sklearn.ensemble import IsolationForest
import numpy as np

# Connect to Prometheus
prom = PrometheusConnect(url="http://localhost:9090")

# Fetch metrics
result = prom.custom_query(
    query='rate(http_requests_total[5m])'
)

# Extract values
values = [float(r['value'][1]) for r in result]

# Train detector
detector = IsolationForest(contamination=0.1)
detector.fit(np.array(values).reshape(-1, 1))

# Detect anomalies
predictions = detector.predict(np.array(values).reshape(-1, 1))
anomalies = [v for v, p in zip(values, predictions) if p == -1]

print(f"Found {len(anomalies)} anomalies")
```

---

## 📚 Lab 2: Log Clustering

**Objective**: Cluster similar log messages

```python
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.cluster import KMeans

logs = [
    "Connection refused to database",
    "Database connection failed", 
    "User login successful",
    "User authenticated",
    "Connection timeout to db",
]

vectorizer = TfidfVectorizer()
X = vectorizer.fit_transform(logs)

kmeans = KMeans(n_clusters=2)
labels = kmeans.fit_predict(X)

for log, label in zip(logs, labels):
    print(f"Cluster {label}: {log}")
```

---

## 📚 Lab 3: Auto-Remediation

**Objective**: Automatically scale on high CPU

```python
# remediation.py
from kubernetes import client, config

config.load_incluster_config()
apps = client.AppsV1Api()

def scale_deployment(name, namespace, replicas):
    body = {"spec": {"replicas": replicas}}
    apps.patch_namespaced_deployment_scale(
        name=name,
        namespace=namespace,
        body=body
    )
    print(f"Scaled {name} to {replicas} replicas")

# Alert handler
def handle_high_cpu(alert):
    deployment = alert['labels']['deployment']
    scale_deployment(deployment, 'default', 5)
```

---

## ✅ Completion Checklist

- [ ] Lab 1: Anomaly detection
- [ ] Lab 2: Log clustering
- [ ] Lab 3: Auto-remediation

