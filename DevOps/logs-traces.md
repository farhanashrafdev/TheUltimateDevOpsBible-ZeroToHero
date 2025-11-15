# Logs & Traces: Complete Observability Guide

## 🎯 Introduction

Logs and traces are essential pillars of observability, providing insights into application behavior, performance, and issues. This guide covers everything you need to implement effective logging and distributed tracing.

### Observability Pillars

1. **Metrics**: Numerical measurements (Prometheus)
2. **Logs**: Event records (ELK, Loki)
3. **Traces**: Request flow (Jaeger, Zipkin)

## 📝 Logging Fundamentals

### Log Levels

**Standard Levels** (from least to most severe):

```
DEBUG → INFO → WARN → ERROR → FATAL
```

**When to Use Each**:

- **DEBUG**: Detailed diagnostic information
  ```python
  logger.debug(f"Processing user {user_id} with data {data}")
  ```

- **INFO**: General informational messages
  ```python
  logger.info(f"User {user_id} logged in successfully")
  ```

- **WARN**: Warning messages for potential issues
  ```python
  logger.warning(f"High memory usage: {memory_percent}%")
  ```

- **ERROR**: Error conditions that don't stop execution
  ```python
  logger.error(f"Failed to connect to database: {error}")
  ```

- **FATAL/CRITICAL**: Critical failures requiring immediate attention
  ```python
  logger.critical(f"System failure: {error}")
  ```

### Structured Logging

**Why Structured Logging?**
- Machine-readable
- Easy to query and filter
- Better correlation
- Consistent format

**JSON Format**:

```python
import json
import logging
from datetime import datetime

class JSONFormatter(logging.Formatter):
    def format(self, record):
        log_entry = {
            "timestamp": datetime.utcnow().isoformat(),
            "level": record.levelname,
            "logger": record.name,
            "message": record.getMessage(),
            "module": record.module,
            "function": record.funcName,
            "line": record.lineno,
        }
        
        # Add extra fields
        if hasattr(record, "request_id"):
            log_entry["request_id"] = record.request_id
        if hasattr(record, "user_id"):
            log_entry["user_id"] = record.user_id
        if hasattr(record, "duration_ms"):
            log_entry["duration_ms"] = record.duration_ms
        
        # Add exception info
        if record.exc_info:
            log_entry["exception"] = self.formatException(record.exc_info)
        
        return json.dumps(log_entry)

# Configure logger
logger = logging.getLogger(__name__)
handler = logging.StreamHandler()
handler.setFormatter(JSONFormatter())
logger.addHandler(handler)
logger.setLevel(logging.INFO)

# Usage
logger.info("Request processed", extra={
    "request_id": "abc123",
    "user_id": "user456",
    "duration_ms": 45,
    "status_code": 200
})
```

**Output**:
```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "INFO",
  "logger": "__main__",
  "message": "Request processed",
  "module": "app",
  "function": "handle_request",
  "line": 42,
  "request_id": "abc123",
  "user_id": "user456",
  "duration_ms": 45,
  "status_code": 200
}
```

### Logging Best Practices

#### 1. Include Context

```python
# Good - With context
logger.info("User login", extra={
    "user_id": user_id,
    "ip_address": request.remote_addr,
    "user_agent": request.headers.get("User-Agent")
})

# Bad - Without context
logger.info("User login")
```

#### 2. Use Appropriate Levels

```python
# Don't use ERROR for warnings
logger.warning("High memory usage")  # Not logger.error

# Don't use INFO for debug info
logger.debug("Variable value: %s", value)  # Not logger.info
```

#### 3. Don't Log Sensitive Data

```python
# Bad
logger.info(f"Password: {password}")

# Good
logger.info("Password reset requested", extra={"user_id": user_id})
```

#### 4. Use Correlation IDs

```python
import uuid
from contextvars import ContextVar

request_id_var: ContextVar[str] = ContextVar('request_id', default=None)

# Middleware
def add_request_id(request):
    request_id = str(uuid.uuid4())
    request_id_var.set(request_id)
    return request_id

# In handlers
logger.info("Processing request", extra={
    "request_id": request_id_var.get()
})
```

## 🔍 Log Aggregation

### ELK Stack (Elasticsearch, Logstash, Kibana)

**Architecture**:
```
Applications → Logstash → Elasticsearch → Kibana
```

#### Elasticsearch

**Storage and search engine**

```yaml
# docker-compose.yml
version: '3.8'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ports:
      - "9200:9200"
    volumes:
      - es_data:/usr/share/elasticsearch/data

volumes:
  es_data:
```

#### Logstash

**Log processing pipeline**

```ruby
# logstash.conf
input {
  file {
    path => "/var/log/app.log"
    start_position => "beginning"
    codec => "json"
  }
  
  beats {
    port => 5044
  }
}

filter {
  if [fields][service] == "api" {
    mutate {
      add_field => { "service_type" => "api" }
    }
  }
  
  date {
    match => [ "timestamp", "ISO8601" ]
  }
  
  grok {
    match => { "message" => "%{COMBINEDAPACHELOG}" }
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "logs-%{+YYYY.MM.dd}"
  }
  
  stdout {
    codec => rubydebug
  }
}
```

#### Kibana

**Visualization and dashboards**

```yaml
# docker-compose.yml
kibana:
  image: docker.elastic.co/kibana/kibana:8.11.0
  ports:
    - "5601:5601"
  environment:
    - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
  depends_on:
    - elasticsearch
```

### Loki (Grafana Labs)

**Lightweight log aggregation**

```yaml
# docker-compose.yml
version: '3.8'
services:
  loki:
    image: grafana/loki:latest
    ports:
      - "3100:3100"
    command: -config.file=/etc/loki/local-config.yaml
  
  promtail:
    image: grafana/promtail:latest
    volumes:
      - /var/log:/var/log:ro
      - ./promtail-config.yml:/etc/promtail/config.yml
    command: -config.file=/etc/promtail/config.yml
```

**Promtail Configuration**:
```yaml
# promtail-config.yml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: system
    static_configs:
      - targets:
          - localhost
        labels:
          job: varlogs
          __path__: /var/log/*.log
```

### Fluentd

**Unified logging layer**

```ruby
# fluentd.conf
<source>
  @type tail
  path /var/log/app.log
  pos_file /var/log/app.log.pos
  tag app.logs
  format json
</source>

<filter app.logs>
  @type record_transformer
  <record>
    hostname "#{Socket.gethostname}"
    environment "#{ENV['ENVIRONMENT']}"
  </record>
</filter>

<match app.logs>
  @type elasticsearch
  host elasticsearch
  port 9200
  index_name app-logs
  type_name _doc
  <buffer>
    @type file
    path /var/log/fluentd-buffers/elasticsearch.buffer
    flush_interval 10s
  </buffer>
</match>
```

## 🔍 Distributed Tracing

### What is Distributed Tracing?

**Tracing** follows a request as it flows through multiple services, showing:
- Which services handled the request
- How long each service took
- Where errors occurred
- Dependencies between services

### OpenTelemetry

**Industry standard for observability**

#### Python Example

```python
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.instrumentation.requests import RequestsInstrumentor
from opentelemetry.instrumentation.flask import FlaskInstrumentor

# Setup
trace.set_tracer_provider(TracerProvider())
tracer = trace.get_tracer(__name__)

# Export to Jaeger
jaeger_exporter = OTLPSpanExporter(
    endpoint="http://jaeger:4317",
    insecure=True
)
span_processor = BatchSpanProcessor(jaeger_exporter)
trace.get_tracer_provider().add_span_processor(span_processor)

# Instrument Flask
FlaskInstrumentor().instrument_app(app)
RequestsInstrumentor().instrument()

# Manual tracing
def process_order(order_id):
    with tracer.start_as_current_span("process_order") as span:
        span.set_attribute("order.id", order_id)
        
        # Call database
        with tracer.start_as_current_span("db.query") as db_span:
            db_span.set_attribute("db.query", "SELECT * FROM orders")
            result = db.get_order(order_id)
        
        # Call payment service
        with tracer.start_as_current_span("payment.process") as payment_span:
            payment_span.set_attribute("payment.amount", result.amount)
            process_payment(order_id)
        
        span.set_attribute("order.status", "completed")
```

#### Node.js Example

```javascript
const { NodeTracerProvider } = require('@opentelemetry/sdk-trace-node');
const { SimpleSpanProcessor } = require('@opentelemetry/sdk-trace-base');
const { JaegerExporter } = require('@opentelemetry/exporter-jaeger');
const { registerInstrumentations } = require('@opentelemetry/instrumentation');
const { HttpInstrumentation } = require('@opentelemetry/instrumentation-http');

const provider = new NodeTracerProvider();
provider.addSpanProcessor(
  new SimpleSpanProcessor(
    new JaegerExporter({
      endpoint: 'http://jaeger:14268/api/traces',
    })
  )
);

registerInstrumentations({
  instrumentations: [new HttpInstrumentation()],
});

provider.register();
```

### Jaeger

**Distributed tracing system**

#### Installation

```bash
# Docker
docker run -d --name jaeger \
  -e COLLECTOR_ZIPKIN_HOST_PORT=:9411 \
  -p 5775:5775/udp \
  -p 6831:6831/udp \
  -p 6832:6832/udp \
  -p 5778:5778 \
  -p 16686:16686 \
  -p 14268:14268 \
  -p 14250:14250 \
  -p 9411:9411 \
  jaegertracing/all-in-one:latest
```

#### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jaeger
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jaeger
  template:
    metadata:
      labels:
        app: jaeger
    spec:
      containers:
      - name: jaeger
        image: jaegertracing/all-in-one:latest
        ports:
        - containerPort: 16686
          name: ui
        - containerPort: 14268
          name: http
        env:
        - name: COLLECTOR_ZIPKIN_HOST_PORT
          value: ":9411"
---
apiVersion: v1
kind: Service
metadata:
  name: jaeger
spec:
  selector:
    app: jaeger
  ports:
  - port: 16686
    targetPort: 16686
    name: ui
  - port: 14268
    targetPort: 14268
    name: http
  type: LoadBalancer
```

### Zipkin

**Alternative tracing system**

```yaml
# docker-compose.yml
version: '3.8'
services:
  zipkin:
    image: openzipkin/zipkin:latest
    ports:
      - "9411:9411"
```

## 📊 Log Analysis

### Common Queries

#### Elasticsearch (Kibana)

```json
// Find errors in last hour
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "level": "ERROR"
          }
        },
        {
          "range": {
            "timestamp": {
              "gte": "now-1h"
            }
          }
        }
      ]
    }
  }
}

// Find slow requests
{
  "query": {
    "range": {
      "duration_ms": {
        "gte": 1000
      }
    }
  },
  "sort": [
    {
      "duration_ms": {
        "order": "desc"
      }
    }
  ]
}
```

#### Loki (LogQL)

```logql
# Count errors
sum(count_over_time({app="myapp"} |= "ERROR" [5m]))

# Top error messages
topk(10, sum by (message) (count_over_time({app="myapp"} |= "ERROR" [1h])))

# Request rate
rate({app="myapp"} | json | __error__="" [5m])

# P95 latency
histogram_quantile(0.95, 
  sum(rate({app="myapp"} | json | duration_ms [5m])) by (le)
)
```

## 🔗 Correlating Logs and Traces

### Trace Context Propagation

```python
from opentelemetry import trace
from opentelemetry.propagate import inject, extract
from opentelemetry.trace.propagation.tracecontext import TraceContextTextMapPropagator

# Extract trace context from incoming request
def handle_request(request):
    carrier = dict(request.headers)
    context = extract(carrier)
    
    tracer = trace.get_tracer(__name__)
    with tracer.start_as_current_span("handle_request", context=context) as span:
        span.set_attribute("http.method", request.method)
        span.set_attribute("http.url", request.url)
        
        # Log with trace ID
        trace_id = format(span.get_span_context().trace_id, '032x')
        logger.info("Processing request", extra={
            "trace_id": trace_id,
            "span_id": format(span.get_span_context().span_id, '016x')
        })
        
        # Process request
        process_request(request)
```

### Unified Observability

```python
# Log with trace context
def log_with_trace(message, level="info"):
    span = trace.get_current_span()
    if span:
        context = span.get_span_context()
        trace_id = format(context.trace_id, '032x')
        span_id = format(context.span_id, '016x')
        
        getattr(logger, level)(message, extra={
            "trace_id": trace_id,
            "span_id": span_id
        })
    else:
        getattr(logger, level)(message)
```

## 🎯 Best Practices

### 1. Structured Logging

```python
# Always use structured logs
logger.info("User action", extra={
    "user_id": user_id,
    "action": "login",
    "ip": request.remote_addr
})
```

### 2. Include Request IDs

```python
# Add request ID to all logs
request_id = str(uuid.uuid4())
logger.info("Request started", extra={"request_id": request_id})
```

### 3. Set Appropriate Levels

```python
# Use correct log levels
logger.debug("Variable value")  # Development
logger.info("User action")       # Normal operation
logger.warning("High usage")    # Potential issue
logger.error("Operation failed") # Error occurred
logger.critical("System down")  # Critical failure
```

### 4. Don't Log Sensitive Data

```python
# Never log passwords, tokens, PII
# Log metadata instead
logger.info("User login", extra={"user_id": user_id})  # Not password
```

### 5. Use Sampling for High Volume

```python
import random

if random.random() < 0.1:  # Sample 10%
    logger.debug("Detailed debug info")
```

### 6. Implement Log Rotation

```python
from logging.handlers import RotatingFileHandler

handler = RotatingFileHandler(
    'app.log',
    maxBytes=10*1024*1024,  # 10MB
    backupCount=5
)
logger.addHandler(handler)
```

## ✅ Mastery Checklist

- [ ] Implement structured logging
- [ ] Set up log aggregation
- [ ] Configure distributed tracing
- [ ] Correlate logs and traces
- [ ] Create log dashboards
- [ ] Set up log alerts
- [ ] Implement log rotation
- [ ] Use appropriate log levels
- [ ] Include correlation IDs
- [ ] Monitor log volume

---

**Next Steps**:
- Learn [Monitoring & Observability](./monitoring-observability.md)
- Explore [On-Call & SRE](./oncall-sre.md)
- Master [Troubleshooting](./troubleshooting-guide.md)

**Remember**: Logs and traces are essential for debugging and understanding system behavior. Use structured logging, implement distributed tracing, and always correlate logs with traces for complete observability.
