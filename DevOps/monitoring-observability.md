# Monitoring & Observability

## 🎯 Three Pillars

### Metrics
- Numerical measurements over time
- Examples: CPU usage, request rate, error rate

### Logs
- Event records
- Examples: Application logs, access logs

### Traces
- Request flow through services
- Examples: Distributed tracing

## 📊 Monitoring Stack

### Prometheus
```yaml
# Prometheus config
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
```

### Grafana
- Visualization and dashboards
- Alerting
- Data source integration

### AlertManager
- Alert routing
- Grouping and deduplication
- Notification channels

## 📝 Key Metrics

### Application Metrics
- Request rate
- Error rate
- Latency (p50, p95, p99)
- Throughput

### Infrastructure Metrics
- CPU usage
- Memory usage
- Disk I/O
- Network traffic

## ✅ Best Practices

- Monitor the four golden signals
- Set up proper alerting
- Use SLIs and SLOs
- Implement dashboards
- Regular review of metrics
- Automate responses

---

**Next**: Learn about logs and traces.

