# Coding Interview Tasks

## 🎯 Common Tasks

### Task 1: Log Parser
Parse log files and extract statistics (IP addresses, status codes, errors).

### Task 2: Configuration Parser
Parse YAML/JSON configuration files and validate structure.

### Task 3: API Client
Create client for REST API with error handling and retries.

### Task 4: Deployment Script
Script to deploy application with health checks and rollback.

### Task 5: Monitoring Script
Script to monitor system resources and alert on thresholds.

## 📝 Example: Log Analyzer

```python
import re
from collections import Counter

def analyze_logs(log_file):
    ips = []
    status_codes = []
    
    with open(log_file, 'r') as f:
        for line in f:
            # Extract IP
            ip_match = re.search(r'\b(\d{1,3}\.){3}\d{1,3}\b', line)
            if ip_match:
                ips.append(ip_match.group())
            
            # Extract status code
            status_match = re.search(r'\s(\d{3})\s', line)
            if status_match:
                status_codes.append(status_match.group(1))
    
    return {
        'total_requests': len(ips),
        'unique_ips': len(set(ips)),
        'top_ips': Counter(ips).most_common(10),
        'status_distribution': Counter(status_codes)
    }
```

## ✅ Preparation Tips

- Practice Python/Bash scripting
- Understand data structures
- Know common algorithms
- Practice problem-solving
- Write clean, readable code
- Test your solutions

---

**Next**: Review cheat sheets.

