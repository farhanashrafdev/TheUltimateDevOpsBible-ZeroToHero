# Logs & Traces

## 🎯 Logging

### Log Levels
- **DEBUG**: Detailed information
- **INFO**: General information
- **WARN**: Warning messages
- **ERROR**: Error conditions
- **FATAL**: Critical failures

### Log Aggregation
- **ELK Stack**: Elasticsearch, Logstash, Kibana
- **Loki**: Log aggregation system
- **Fluentd**: Log collector

## 📝 Logging Best Practices

### Structured Logging
```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "INFO",
  "service": "api",
  "message": "Request processed",
  "request_id": "abc123",
  "duration_ms": 45
}
```

### Centralized Logging
```yaml
# Fluentd config
<source>
  @type tail
  path /var/log/app.log
  pos_file /var/log/app.log.pos
  tag app.logs
</source>

<match app.logs>
  @type elasticsearch
  host elasticsearch
  port 9200
</match>
```

## 🔍 Distributed Tracing

### OpenTelemetry
```python
from opentelemetry import trace
from opentelemetry.exporter.jaeger import JaegerExporter

tracer = trace.get_tracer(__name__)

with tracer.start_as_current_span("operation"):
    # Your code here
    pass
```

### Jaeger
- Distributed tracing system
- Request flow visualization
- Performance analysis

## ✅ Best Practices

- Use structured logging
- Include context (request IDs)
- Set appropriate log levels
- Implement log rotation
- Use distributed tracing
- Correlate logs and traces

---

**Next**: Learn on-call and SRE practices.

