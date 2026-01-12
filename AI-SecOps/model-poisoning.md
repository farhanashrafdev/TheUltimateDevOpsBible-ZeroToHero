# Model Poisoning

## 🎯 Introduction

Model poisoning attacks corrupt ML models by manipulating training data or the training process.

## 📚 Attack Types

```
┌─────────────────────────────────────────────────────────────────────┐
│                     Model Poisoning Attacks                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Data Poisoning               Model Poisoning                       │
│  ├── Label flipping           ├── Trojan/backdoor                   │
│  ├── Clean-label attacks      ├── Gradient manipulation            │
│  └── Data injection           └── Fine-tuning attacks              │
│                                                                      │
│  Attack Goals:                                                       │
│  ├── Reduce model accuracy (availability)                           │
│  ├── Cause specific misclassifications (integrity)                 │
│  └── Embed backdoors for later exploitation                         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 🔧 Defense Strategies

### Data Validation

```python
class DataValidator:
    def __init__(self, statistical_checks=True, outlier_detection=True):
        self.checks_enabled = statistical_checks
        self.outlier_detection = outlier_detection
    
    def validate_batch(self, data, labels):
        """Validate training data batch."""
        issues = []
        
        # Check for statistical anomalies
        if self.checks_enabled:
            if self.detect_distribution_shift(data):
                issues.append("Distribution shift detected")
        
        # Check for outliers
        if self.outlier_detection:
            outliers = self.find_outliers(data)
            if len(outliers) > 0.05 * len(data):
                issues.append(f"High outlier rate: {len(outliers)}")
        
        # Check label consistency
        if self.detect_label_noise(data, labels):
            issues.append("Potential label noise")
        
        return len(issues) == 0, issues
```

### Robust Training

```python
# Techniques for poison-resistant training
import torch

def robust_training(model, data_loader, epochs):
    optimizer = torch.optim.Adam(model.parameters())
    
    for epoch in range(epochs):
        for batch_data, batch_labels in data_loader:
            # Gradient clipping to prevent extreme updates
            loss = compute_loss(model, batch_data, batch_labels)
            loss.backward()
            
            torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
            
            optimizer.step()
            optimizer.zero_grad()
```

## ✅ Defenses

1. **Data Provenance**: Track data sources
2. **Anomaly Detection**: Flag suspicious samples
3. **Robust Aggregation**: Filter outlier gradients
4. **Differential Privacy**: Limit influence per sample
5. **Model Validation**: Test against known attacks

---

**Next**: Learn about [Prompt Injection Defense](./prompt-injection-defense.md).

