# AIOps Interview Questions

## 🎯 Fundamentals

**Q: What is AIOps?**

**A:** AIOps (Artificial Intelligence for IT Operations) uses ML to:
- Automate IT operations
- Detect anomalies before users notice
- Correlate events across systems
- Suggest or execute remediation

**Q: What are the key components of an AIOps platform?**

**A:**
1. **Data Ingestion**: Metrics, logs, traces
2. **Anomaly Detection**: ML models for unusual behavior
3. **Event Correlation**: Connect related events
4. **Root Cause Analysis**: Identify problem source
5. **Automated Remediation**: Self-healing actions

## 📊 Anomaly Detection

**Q: How do you detect anomalies in time-series data?**

**A:**
- **Statistical**: Z-score, IQR, ARIMA
- **ML-based**: Isolation Forest, One-class SVM
- **Deep Learning**: LSTM autoencoders, Transformers
- **Seasonal**: STL decomposition + threshold

**Q: What's the difference between supervised and unsupervised anomaly detection?**

| Supervised | Unsupervised |
|------------|--------------|
| Needs labeled data | No labels needed |
| Detects known patterns | Detects unknown patterns |
| Classification problem | Clustering/density based |

## 🔧 Automated Remediation

**Q: How do you safely implement auto-remediation?**

**A:**
1. Start with low-risk actions (restart, scale)
2. Implement safeguards (cooldowns, limits)
3. Require human approval for risky actions
4. Comprehensive logging
5. Easy rollback mechanism

**Q: Design an auto-scaling system using ML.**

**A:**
- Collect historical metrics (CPU, requests, latency)
- Train model to predict future load
- Proactively scale before demand spike
- Continuously retrain on new data
- Fall back to reactive scaling if prediction fails

## 🎯 Scenario Questions

**Q: How would you reduce alert fatigue?**

**A:**
1. ML-based alert correlation
2. Automatic severity adjustment
3. Deduplication and grouping
4. Context-aware routing
5. Feedback loops (was alert actionable?)

---

**Next**: Return to [Interview Overview](./README.md).

