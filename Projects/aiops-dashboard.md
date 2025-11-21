# Project: AIOps Anomaly Detection Dashboard

## 🎯 Objective
Build a dashboard that detects anomalies in system metrics using Machine Learning.

## 🛠️ Tech Stack
- **Metrics**: Prometheus
- **ML**: Python (Prophet / Scikit-learn)
- **Visualization**: Grafana
- **App**: Streamlit (optional for custom UI)

## 📝 Requirements

1. **Data Collection**:
   - Scrape metrics from a running cluster.
   - Store in Prometheus.

2. **Anomaly Detection Engine**:
   - Python script queries Prometheus.
   - Trains model on historical data.
   - Detects current anomalies.

3. **Visualization**:
   - Push anomaly scores back to Prometheus (or display in Streamlit).
   - Grafana dashboard showing "Actual vs Predicted".

## 🚀 Steps

### 1. Setup Monitoring
- Deploy Prometheus and Node Exporter.
- Generate load (stress test) to create metric history.

### 2. Build ML Model
- Export data to CSV.
- Train Prophet model.
- Predict next hour.

### 3. Real-time Detection
- Create a loop: Query -> Predict -> Alert.
- If `Actual > Predicted + Threshold`, fire alert.

## 🏆 Definition of Done
- [ ] System collects metrics in real-time.
- [ ] ML model predicts expected load.
- [ ] Dashboard highlights anomalies.
- [ ] Alert fires when anomaly detected.

---

**Next**: Explore [Labs](../Hands-On/overview.md) for bite-sized practice.
