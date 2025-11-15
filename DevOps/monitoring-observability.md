# Monitoring & Observability: Complete Guide

## 🎯 Introduction

Monitoring and observability are critical for understanding system health, performance, and behavior. This guide covers the three pillars of observability: metrics, logs, and traces, along with tools and best practices.

### Observability vs Monitoring

**Monitoring**: 
- What you know you need to watch
- Predefined metrics and alerts
- Reactive approach

**Observability**:
- Ability to understand system state from outputs
- Exploratory approach
- Answers questions you didn't know to ask

### Three Pillars of Observability

```
┌─────────────────────────────────────┐
│         Observability                │
│                                     │
│  ┌──────────┐  ┌──────────┐        │
│  │ Metrics  │  │  Logs    │        │
│  │          │  │          │        │
│  └──────────┘  └──────────┘        │
│                                     │
│  ┌──────────┐                      │
│  │ Traces   │                      │
│  │          │                      │
│  └──────────┘                      │
└─────────────────────────────────────┘
```

## 📊 Metrics

### What are Metrics?

**Metrics** are numerical measurements collected over time, representing system behavior and performance.

### Four Golden Signals

1. **Latency**: Time to serve a request
2. **Traffic**: Demand on the system
3. **Errors**: Rate of failed requests
4. **Saturation**: Resource utilization

### Prometheus

**Industry-standard metrics collection**

#### Installation

```bash
# Docker
docker run -d \
  --name prometheus \
  -p 9090:9090 \
  -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus:latest
```

#### Configuration

```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    cluster: 'production'
    environment: 'prod'

rule_files:
  - "alerts.yml"

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__
  
  - job_name: 'kubernetes-nodes'
    kubernetes_sd_configs:
      - role: node
    relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
  
  - job_name: 'kubernetes-services'
    kubernetes_sd_configs:
      - role: service
    metrics_path: /probe
    params:
      module: [http_2xx]
    relabel_configs:
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_probe]
        action: keep
        regex: true
```

#### Alerting Rules

```yaml
# alerts.yml
groups:
  - name: instance_alerts
    interval: 30s
    rules:
      - alert: InstanceDown
        expr: up == 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Instance {{ $labels.instance }} down"
          description: "{{ $labels.instance }} has been down for more than 5 minutes."
      
      - alert: HighCPUUsage
        expr: 100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage on {{ $labels.instance }}"
          description: "CPU usage is above 80% for 5 minutes."
      
      - alert: HighMemoryUsage
        expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 85
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage on {{ $labels.instance }}"
          description: "Memory usage is above 85% for 5 minutes."
      
      - alert: DiskSpaceLow
        expr: (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100 < 15
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Low disk space on {{ $labels.instance }}"
          description: "Disk space is below 15%."
```

### PromQL (Prometheus Query Language)

**Common Queries**:

```promql
# CPU usage percentage
100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory usage percentage
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Request rate
rate(http_requests_total[5m])

# Error rate
rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m])

# P95 latency
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# Top 10 by CPU
topk(10, rate(container_cpu_usage_seconds_total[5m]))
```

### Application Metrics

#### Python (Prometheus Client)

```python
from prometheus_client import Counter, Histogram, Gauge, start_http_server
import time

# Define metrics
REQUEST_COUNT = Counter('http_requests_total', 'Total HTTP requests', ['method', 'endpoint', 'status'])
REQUEST_DURATION = Histogram('http_request_duration_seconds', 'HTTP request duration', ['method', 'endpoint'])
ACTIVE_CONNECTIONS = Gauge('active_connections', 'Number of active connections')

# Start metrics server
start_http_server(8000)

# Use in application
def handle_request(method, endpoint):
    start_time = time.time()
    
    try:
        # Process request
        result = process_request()
        
        # Record metrics
        REQUEST_COUNT.labels(method=method, endpoint=endpoint, status='200').inc()
        REQUEST_DURATION.labels(method=method, endpoint=endpoint).observe(time.time() - start_time)
        ACTIVE_CONNECTIONS.inc()
        
        return result
    except Exception as e:
        REQUEST_COUNT.labels(method=method, endpoint=endpoint, status='500').inc()
        raise
    finally:
        ACTIVE_CONNECTIONS.dec()
```

#### Node.js (Prometheus Client)

```javascript
const client = require('prom-client');

// Create metrics
const httpRequestDuration = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.1, 0.5, 1, 2, 5]
});

const httpRequestTotal = new client.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status_code']
});

// Middleware
app.use((req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    httpRequestDuration.observe(
      { method: req.method, route: req.route?.path || req.path, status_code: res.statusCode },
      duration
    );
    httpRequestTotal.inc({ method: req.method, route: req.route?.path || req.path, status_code: res.statusCode });
  });
  
  next();
});

// Metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', client.register.contentType);
  res.end(await client.register.metrics());
});
```

## 📊 Grafana

**Visualization and dashboards**

### Installation

```yaml
# docker-compose.yml
version: '3.8'
services:
  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning

volumes:
  grafana_data:
```

### Dashboard Example

```json
{
  "dashboard": {
    "title": "Application Metrics",
    "panels": [
      {
        "title": "Request Rate",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])",
            "legendFormat": "{{method}} {{endpoint}}"
          }
        ]
      },
      {
        "title": "Error Rate",
        "targets": [
          {
            "expr": "rate(http_requests_total{status=~\"5..\"}[5m])",
            "legendFormat": "Errors"
          }
        ]
      },
      {
        "title": "P95 Latency",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))",
            "legendFormat": "P95"
          }
        ]
      }
    ]
  }
}
```

## 🎯 SLIs, SLOs, and SLAs

### Service Level Indicators (SLIs)

**Measurable aspects of service quality**

```yaml
# Example SLIs
availability: successful_requests / total_requests
latency: p95_request_duration
error_rate: error_requests / total_requests
throughput: requests_per_second
```

### Service Level Objectives (SLOs)

**Target values for SLIs**

```yaml
# Example SLOs
availability: 99.9% (3 nines)
latency_p95: < 200ms
error_rate: < 0.1%
throughput: > 1000 req/s
```

### Service Level Agreements (SLAs)

**Contractual commitments**

```yaml
# Example SLA
availability: 99.9% uptime
# If violated: Service credits or penalties
```

### Error Budgets

**Acceptable failure rate**

```yaml
# 99.9% availability = 0.1% error budget
# Over 30 days: ~43 minutes of downtime allowed
```

## 📊 Key Metrics to Monitor

### Application Metrics

```promql
# Request rate
rate(http_requests_total[5m])

# Error rate
rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m])

# Latency (p50, p95, p99)
histogram_quantile(0.50, rate(http_request_duration_seconds_bucket[5m]))
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))

# Throughput
sum(rate(http_requests_total[5m]))
```

### Infrastructure Metrics

```promql
# CPU usage
100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory usage
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Disk usage
(1 - (node_filesystem_avail_bytes / node_filesystem_size_bytes)) * 100

# Network I/O
rate(node_network_receive_bytes_total[5m])
rate(node_network_transmit_bytes_total[5m])
```

### Kubernetes Metrics

```promql
# Pod CPU usage
sum(rate(container_cpu_usage_seconds_total[5m])) by (pod, namespace)

# Pod memory usage
sum(container_memory_usage_bytes) by (pod, namespace)

# Pod restarts
rate(kube_pod_container_status_restarts_total[5m])

# Node CPU
100 - (avg by(node) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

## 🔔 Alerting

### AlertManager

**Alert routing and management**

```yaml
# alertmanager.yml
global:
  resolve_timeout: 5m
  slack_api_url: 'https://hooks.slack.com/services/YOUR/WEBHOOK/URL'

route:
  group_by: ['alertname', 'cluster', 'service']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h
  receiver: 'slack-notifications'
  routes:
    - match:
        severity: critical
      receiver: 'pagerduty'
    - match:
        severity: warning
      receiver: 'slack-notifications'

receivers:
  - name: 'slack-notifications'
    slack_configs:
      - channel: '#alerts'
        title: '{{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'
  
  - name: 'pagerduty'
    pagerduty_configs:
      - service_key: 'YOUR_PAGERDUTY_KEY'
```

## ✅ Best Practices

### 1. Monitor the Four Golden Signals

- Latency
- Traffic
- Errors
- Saturation

### 2. Set Up Proper Alerting

```yaml
# Alert on symptoms, not causes
# Use SLO-based alerting
# Avoid alert fatigue
```

### 3. Use SLIs and SLOs

```yaml
# Define SLIs
# Set SLOs
# Track error budgets
```

### 4. Implement Dashboards

```yaml
# Create dashboards for:
# - System overview
# - Application metrics
# - Business metrics
# - SLO tracking
```

### 5. Regular Review

```yaml
# Review metrics weekly
# Update dashboards
# Refine alerts
# Analyze trends
```

### 6. Automate Responses

```yaml
# Auto-scaling based on metrics
# Auto-remediation
# Runbook automation
```

## ✅ Mastery Checklist

- [ ] Set up Prometheus
- [ ] Configure Grafana dashboards
- [ ] Define SLIs and SLOs
- [ ] Implement application metrics
- [ ] Set up alerting
- [ ] Monitor golden signals
- [ ] Create runbooks
- [ ] Track error budgets
- [ ] Review metrics regularly
- [ ] Automate responses

---

**Next Steps**:
- Learn [Logs & Traces](./logs-traces.md)
- Explore [On-Call & SRE](./oncall-sre.md)
- Master [Production Practices](./production-practices.md)

**Remember**: Monitoring is about understanding your system. Start with the four golden signals, implement proper alerting, and continuously refine based on what you learn. Good observability enables fast incident response and continuous improvement.
