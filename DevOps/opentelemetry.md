# OpenTelemetry: Complete Observability Guide

## 🎯 Introduction

OpenTelemetry (OTel) is an open-source observability framework for generating, collecting, and exporting telemetry data (traces, metrics, logs). It's the merger of OpenTracing and OpenCensus, becoming the industry standard for observability instrumentation.

### The Three Pillars of Observability

```
┌─────────────────────────────────────────────────────────────────┐
│                     Observability Pillars                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │    TRACES    │  │   METRICS    │  │     LOGS     │          │
│  │              │  │              │  │              │          │
│  │ Request flow │  │ Aggregated   │  │ Discrete     │          │
│  │ across       │  │ measurements │  │ events with  │          │
│  │ services     │  │ over time    │  │ context      │          │
│  │              │  │              │  │              │          │
│  │ Span A       │  │ Counter      │  │ [INFO] ...   │          │
│  │  └─Span B    │  │ Gauge        │  │ [ERROR] ...  │          │
│  │    └─Span C  │  │ Histogram    │  │ [DEBUG] ...  │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│                                                                  │
│  "What happened?"  "How is it     "What was                     │
│                    performing?"    recorded?"                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## 🏗️ OpenTelemetry Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                    OpenTelemetry Architecture                     │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │   Service   │  │   Service   │  │   Service   │              │
│  │      A      │  │      B      │  │      C      │              │
│  │ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ │              │
│  │ │OTel SDK │ │  │ │OTel SDK │ │  │ │OTel SDK │ │              │
│  │ └────┬────┘ │  │ └────┬────┘ │  │ └────┬────┘ │              │
│  └──────┼──────┘  └──────┼──────┘  └──────┼──────┘              │
│         │                │                │                      │
│         ▼                ▼                ▼                      │
│  ┌────────────────────────────────────────────────────┐         │
│  │              OpenTelemetry Collector               │         │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐        │         │
│  │  │Receivers │─▶│Processors│─▶│Exporters │        │         │
│  │  └──────────┘  └──────────┘  └──────────┘        │         │
│  └────────────────────────────────────────────────────┘         │
│                              │                                   │
│         ┌────────────────────┼────────────────────┐             │
│         ▼                    ▼                    ▼             │
│  ┌─────────────┐     ┌─────────────┐      ┌─────────────┐      │
│  │   Jaeger    │     │ Prometheus  │      │    Loki     │      │
│  │  (Traces)   │     │  (Metrics)  │      │   (Logs)    │      │
│  └─────────────┘     └─────────────┘      └─────────────┘      │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

## 📦 OpenTelemetry Collector

### Installation

```bash
# Kubernetes with Helm
helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
helm repo update

# Install as DaemonSet (agent mode)
helm install otel-collector open-telemetry/opentelemetry-collector \
  --namespace observability --create-namespace \
  --set mode=daemonset \
  -f otel-values.yaml

# Install as Deployment (gateway mode)
helm install otel-gateway open-telemetry/opentelemetry-collector \
  --namespace observability \
  --set mode=deployment \
  --set replicaCount=3 \
  -f otel-gateway-values.yaml
```

### Collector Configuration

```yaml
# otel-collector-config.yaml
receivers:
  # OTLP receiver - standard OTel protocol
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318
  
  # Prometheus receiver - scrape Prometheus metrics
  prometheus:
    config:
      scrape_configs:
        - job_name: 'kubernetes-pods'
          kubernetes_sd_configs:
            - role: pod
          relabel_configs:
            - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
              action: keep
              regex: true
  
  # Host metrics receiver
  hostmetrics:
    collection_interval: 30s
    scrapers:
      cpu:
      memory:
      disk:
      network:
  
  # Kubernetes cluster receiver
  k8s_cluster:
    collection_interval: 30s
    node_conditions_to_report:
      - Ready
      - MemoryPressure
    allocatable_types_to_report:
      - cpu
      - memory

processors:
  # Batch processor - groups data for efficient export
  batch:
    timeout: 10s
    send_batch_size: 1000
    send_batch_max_size: 2000
  
  # Memory limiter - prevents OOM
  memory_limiter:
    check_interval: 1s
    limit_mib: 1000
    spike_limit_mib: 200
  
  # Resource processor - add/modify attributes
  resource:
    attributes:
      - key: environment
        value: production
        action: upsert
      - key: k8s.cluster.name
        value: prod-cluster
        action: upsert
  
  # Attributes processor - modify span/metric attributes
  attributes:
    actions:
      - key: db.password
        action: delete
      - key: http.request.header.authorization
        action: delete
  
  # Tail sampling - intelligent trace sampling
  tail_sampling:
    decision_wait: 10s
    num_traces: 100000
    policies:
      # Always sample errors
      - name: errors
        type: status_code
        status_code:
          status_codes: [ERROR]
      # Sample slow requests
      - name: slow-requests
        type: latency
        latency:
          threshold_ms: 1000
      # Sample 10% of everything else
      - name: probabilistic
        type: probabilistic
        probabilistic:
          sampling_percentage: 10

exporters:
  # Jaeger exporter
  jaeger:
    endpoint: jaeger-collector:14250
    tls:
      insecure: true
  
  # Prometheus exporter
  prometheus:
    endpoint: 0.0.0.0:8889
    namespace: otel
    const_labels:
      environment: production
  
  # OTLP exporter (to another collector or backend)
  otlp:
    endpoint: tempo:4317
    tls:
      insecure: true
  
  # Loki exporter for logs
  loki:
    endpoint: http://loki:3100/loki/api/v1/push
    labels:
      attributes:
        service.name: "service_name"
        severity: "severity"
  
  # AWS X-Ray
  awsxray:
    region: us-east-1
    
  # Debug exporter (for testing)
  debug:
    verbosity: detailed

extensions:
  health_check:
    endpoint: 0.0.0.0:13133
  pprof:
    endpoint: 0.0.0.0:1777
  zpages:
    endpoint: 0.0.0.0:55679

service:
  extensions: [health_check, pprof, zpages]
  pipelines:
    traces:
      receivers: [otlp]
      processors: [memory_limiter, batch, resource, attributes, tail_sampling]
      exporters: [jaeger, otlp]
    
    metrics:
      receivers: [otlp, prometheus, hostmetrics, k8s_cluster]
      processors: [memory_limiter, batch, resource]
      exporters: [prometheus]
    
    logs:
      receivers: [otlp]
      processors: [memory_limiter, batch, resource, attributes]
      exporters: [loki]
```

## 🐍 Python Instrumentation

### Auto-Instrumentation

```bash
# Install packages
pip install opentelemetry-distro opentelemetry-exporter-otlp
opentelemetry-bootstrap -a install

# Run with auto-instrumentation
opentelemetry-instrument \
  --traces_exporter otlp \
  --metrics_exporter otlp \
  --logs_exporter otlp \
  --exporter_otlp_endpoint http://localhost:4317 \
  --service_name my-python-service \
  python app.py
```

### Manual Instrumentation

```python
# app.py
from opentelemetry import trace, metrics
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.sdk.metrics import MeterProvider
from opentelemetry.sdk.metrics.export import PeriodicExportingMetricReader
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.exporter.otlp.proto.grpc.metric_exporter import OTLPMetricExporter
from opentelemetry.sdk.resources import Resource, SERVICE_NAME
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor
from flask import Flask
import requests

# Configure resource
resource = Resource(attributes={
    SERVICE_NAME: "user-service",
    "service.version": "1.0.0",
    "deployment.environment": "production"
})

# Configure tracer
tracer_provider = TracerProvider(resource=resource)
tracer_provider.add_span_processor(
    BatchSpanProcessor(OTLPSpanExporter(endpoint="localhost:4317", insecure=True))
)
trace.set_tracer_provider(tracer_provider)
tracer = trace.get_tracer(__name__)

# Configure metrics
metric_reader = PeriodicExportingMetricReader(
    OTLPMetricExporter(endpoint="localhost:4317", insecure=True),
    export_interval_millis=60000
)
meter_provider = MeterProvider(resource=resource, metric_readers=[metric_reader])
metrics.set_meter_provider(meter_provider)
meter = metrics.get_meter(__name__)

# Create metrics
request_counter = meter.create_counter(
    name="http_requests_total",
    description="Total HTTP requests",
    unit="1"
)

request_duration = meter.create_histogram(
    name="http_request_duration_seconds",
    description="HTTP request duration",
    unit="s"
)

active_users = meter.create_up_down_counter(
    name="active_users",
    description="Number of active users"
)

# Flask app
app = Flask(__name__)
FlaskInstrumentor().instrument_app(app)
RequestsInstrumentor().instrument()

@app.route('/users/<user_id>')
def get_user(user_id):
    # Custom span
    with tracer.start_as_current_span("get_user_from_db") as span:
        span.set_attribute("user.id", user_id)
        
        # Simulate DB query
        user = fetch_user_from_db(user_id)
        
        if not user:
            span.set_status(trace.Status(trace.StatusCode.ERROR, "User not found"))
            span.add_event("user_not_found", {"user_id": user_id})
            return {"error": "Not found"}, 404
        
        span.add_event("user_found", {"user_id": user_id, "name": user["name"]})
        
    # Record metrics
    request_counter.add(1, {"endpoint": "/users", "method": "GET", "status": "200"})
    
    return user

def fetch_user_from_db(user_id):
    with tracer.start_as_current_span("db_query") as span:
        span.set_attribute("db.system", "postgresql")
        span.set_attribute("db.statement", "SELECT * FROM users WHERE id = ?")
        # Simulate query
        return {"id": user_id, "name": "John Doe"}

if __name__ == '__main__':
    app.run(port=8080)
```

## ☕ Java Instrumentation

### Auto-Instrumentation with Agent

```bash
# Download the agent
curl -L -o opentelemetry-javaagent.jar \
  https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases/latest/download/opentelemetry-javaagent.jar

# Run with agent
java -javaagent:opentelemetry-javaagent.jar \
  -Dotel.service.name=order-service \
  -Dotel.exporter.otlp.endpoint=http://localhost:4317 \
  -Dotel.traces.exporter=otlp \
  -Dotel.metrics.exporter=otlp \
  -Dotel.logs.exporter=otlp \
  -jar myapp.jar
```

### Spring Boot Configuration

```yaml
# application.yaml
otel:
  service:
    name: order-service
  exporter:
    otlp:
      endpoint: http://otel-collector:4317
  traces:
    exporter: otlp
  metrics:
    exporter: otlp
  logs:
    exporter: otlp

management:
  tracing:
    sampling:
      probability: 1.0
```

```java
// Manual instrumentation
import io.opentelemetry.api.GlobalOpenTelemetry;
import io.opentelemetry.api.trace.Span;
import io.opentelemetry.api.trace.Tracer;
import io.opentelemetry.api.metrics.Meter;
import io.opentelemetry.api.metrics.LongCounter;
import io.opentelemetry.context.Scope;

@Service
public class OrderService {
    
    private final Tracer tracer = GlobalOpenTelemetry.getTracer("order-service");
    private final Meter meter = GlobalOpenTelemetry.getMeter("order-service");
    private final LongCounter ordersCreated = meter
        .counterBuilder("orders_created_total")
        .setDescription("Total orders created")
        .build();
    
    public Order createOrder(CreateOrderRequest request) {
        Span span = tracer.spanBuilder("createOrder")
            .setAttribute("order.customer_id", request.getCustomerId())
            .startSpan();
        
        try (Scope scope = span.makeCurrent()) {
            // Validate order
            validateOrder(request);
            
            // Save to database
            Order order = saveOrder(request);
            
            span.setAttribute("order.id", order.getId());
            span.addEvent("order_created");
            
            // Record metric
            ordersCreated.add(1, 
                Attributes.of(
                    AttributeKey.stringKey("status"), "success",
                    AttributeKey.stringKey("customer_type"), request.getCustomerType()
                )
            );
            
            return order;
            
        } catch (Exception e) {
            span.recordException(e);
            span.setStatus(StatusCode.ERROR, e.getMessage());
            throw e;
        } finally {
            span.end();
        }
    }
    
    private void validateOrder(CreateOrderRequest request) {
        Span span = tracer.spanBuilder("validateOrder").startSpan();
        try (Scope scope = span.makeCurrent()) {
            // Validation logic
        } finally {
            span.end();
        }
    }
}
```

## 🟢 Node.js Instrumentation

### Auto-Instrumentation

```javascript
// tracing.js - Import FIRST before any other imports
const { NodeSDK } = require('@opentelemetry/sdk-node');
const { getNodeAutoInstrumentations } = require('@opentelemetry/auto-instrumentations-node');
const { OTLPTraceExporter } = require('@opentelemetry/exporter-trace-otlp-grpc');
const { OTLPMetricExporter } = require('@opentelemetry/exporter-metrics-otlp-grpc');
const { PeriodicExportingMetricReader } = require('@opentelemetry/sdk-metrics');
const { Resource } = require('@opentelemetry/resources');
const { SemanticResourceAttributes } = require('@opentelemetry/semantic-conventions');

const sdk = new NodeSDK({
  resource: new Resource({
    [SemanticResourceAttributes.SERVICE_NAME]: 'api-service',
    [SemanticResourceAttributes.SERVICE_VERSION]: '1.0.0',
    [SemanticResourceAttributes.DEPLOYMENT_ENVIRONMENT]: 'production',
  }),
  traceExporter: new OTLPTraceExporter({
    url: 'http://localhost:4317',
  }),
  metricReader: new PeriodicExportingMetricReader({
    exporter: new OTLPMetricExporter({
      url: 'http://localhost:4317',
    }),
    exportIntervalMillis: 60000,
  }),
  instrumentations: [
    getNodeAutoInstrumentations({
      '@opentelemetry/instrumentation-fs': { enabled: false },
    }),
  ],
});

sdk.start();

process.on('SIGTERM', () => {
  sdk.shutdown()
    .then(() => console.log('Tracing terminated'))
    .catch((error) => console.log('Error terminating tracing', error))
    .finally(() => process.exit(0));
});
```

```javascript
// app.js
require('./tracing'); // Must be first!

const express = require('express');
const { trace, metrics, context, SpanStatusCode } = require('@opentelemetry/api');

const app = express();
const tracer = trace.getTracer('api-service');
const meter = metrics.getMeter('api-service');

// Create metrics
const requestCounter = meter.createCounter('http_requests_total', {
  description: 'Total HTTP requests',
});

const requestDuration = meter.createHistogram('http_request_duration_ms', {
  description: 'HTTP request duration in milliseconds',
});

app.get('/api/products/:id', async (req, res) => {
  const startTime = Date.now();
  
  const span = tracer.startSpan('getProduct', {
    attributes: {
      'product.id': req.params.id,
      'http.method': 'GET',
    },
  });
  
  try {
    const product = await context.with(
      trace.setSpan(context.active(), span),
      () => fetchProduct(req.params.id)
    );
    
    if (!product) {
      span.setStatus({ code: SpanStatusCode.ERROR, message: 'Product not found' });
      requestCounter.add(1, { endpoint: '/api/products', status: '404' });
      return res.status(404).json({ error: 'Not found' });
    }
    
    span.addEvent('product_fetched', { 'product.name': product.name });
    requestCounter.add(1, { endpoint: '/api/products', status: '200' });
    res.json(product);
    
  } catch (error) {
    span.recordException(error);
    span.setStatus({ code: SpanStatusCode.ERROR, message: error.message });
    requestCounter.add(1, { endpoint: '/api/products', status: '500' });
    res.status(500).json({ error: 'Internal error' });
    
  } finally {
    span.end();
    requestDuration.record(Date.now() - startTime, { endpoint: '/api/products' });
  }
});

async function fetchProduct(id) {
  const span = tracer.startSpan('fetchProductFromDB', {
    attributes: {
      'db.system': 'postgresql',
      'db.operation': 'SELECT',
    },
  });
  
  try {
    // Simulate DB query
    await new Promise(resolve => setTimeout(resolve, 50));
    return { id, name: 'Widget', price: 29.99 };
  } finally {
    span.end();
  }
}

app.listen(3000, () => console.log('Server running on port 3000'));
```

## ☸️ Kubernetes Deployment

### Collector as DaemonSet

```yaml
# otel-collector-daemonset.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: otel-collector-agent
  namespace: observability
spec:
  selector:
    matchLabels:
      app: otel-collector-agent
  template:
    metadata:
      labels:
        app: otel-collector-agent
    spec:
      containers:
      - name: otel-collector
        image: otel/opentelemetry-collector-contrib:0.91.0
        args:
          - "--config=/conf/otel-agent-config.yaml"
        ports:
          - containerPort: 4317  # OTLP gRPC
          - containerPort: 4318  # OTLP HTTP
          - containerPort: 8888  # Prometheus metrics
        env:
          - name: K8S_NODE_NAME
            valueFrom:
              fieldRef:
                fieldPath: spec.nodeName
          - name: K8S_POD_IP
            valueFrom:
              fieldRef:
                fieldPath: status.podIP
        resources:
          limits:
            cpu: 500m
            memory: 512Mi
          requests:
            cpu: 100m
            memory: 128Mi
        volumeMounts:
          - name: config
            mountPath: /conf
      volumes:
        - name: config
          configMap:
            name: otel-agent-config
---
apiVersion: v1
kind: Service
metadata:
  name: otel-collector
  namespace: observability
spec:
  ports:
    - name: otlp-grpc
      port: 4317
      targetPort: 4317
    - name: otlp-http
      port: 4318
      targetPort: 4318
  selector:
    app: otel-collector-agent
```

### Auto-Instrumentation with Operator

```yaml
# Install OTel Operator
# helm install opentelemetry-operator open-telemetry/opentelemetry-operator

# Auto-instrumentation configuration
apiVersion: opentelemetry.io/v1alpha1
kind: Instrumentation
metadata:
  name: auto-instrumentation
  namespace: production
spec:
  exporter:
    endpoint: http://otel-collector:4317
  propagators:
    - tracecontext
    - baggage
  sampler:
    type: parentbased_traceidratio
    argument: "0.25"
  python:
    env:
      - name: OTEL_PYTHON_LOGGING_AUTO_INSTRUMENTATION_ENABLED
        value: "true"
  java:
    image: ghcr.io/open-telemetry/opentelemetry-operator/autoinstrumentation-java:latest
  nodejs:
    image: ghcr.io/open-telemetry/opentelemetry-operator/autoinstrumentation-nodejs:latest
---
# Annotate pods for auto-instrumentation
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-python-app
  namespace: production
spec:
  template:
    metadata:
      annotations:
        instrumentation.opentelemetry.io/inject-python: "true"
    spec:
      containers:
      - name: app
        image: my-python-app:latest
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-java-app
  namespace: production
spec:
  template:
    metadata:
      annotations:
        instrumentation.opentelemetry.io/inject-java: "true"
    spec:
      containers:
      - name: app
        image: my-java-app:latest
```

## 📊 Backends & Visualization

### Grafana Stack (Tempo + Prometheus + Loki)

```yaml
# Grafana datasources
apiVersion: 1
datasources:
  - name: Tempo
    type: tempo
    access: proxy
    url: http://tempo:3100
    jsonData:
      tracesToLogs:
        datasourceUid: loki
        tags: ['service.name', 'trace_id']
      serviceMap:
        datasourceUid: prometheus
      nodeGraph:
        enabled: true
      
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    jsonData:
      exemplarTraceIdDestinations:
        - datasourceUid: tempo
          name: trace_id
      
  - name: Loki
    type: loki
    access: proxy
    url: http://loki:3100
    jsonData:
      derivedFields:
        - datasourceUid: tempo
          matcherRegex: "trace_id=(\\w+)"
          name: TraceID
          url: '$${__value.raw}'
```

## ✅ Best Practices

### Instrumentation
- [ ] Use semantic conventions for attributes
- [ ] Set appropriate resource attributes
- [ ] Use context propagation properly
- [ ] Don't create spans for trivial operations
- [ ] Add meaningful events and attributes

### Sampling
- [ ] Implement tail-based sampling for production
- [ ] Always sample errors and slow requests
- [ ] Use probabilistic sampling for high-volume services
- [ ] Configure parent-based sampling

### Security
- [ ] Sanitize sensitive data in attributes
- [ ] Use TLS for exporter connections
- [ ] Implement proper access controls
- [ ] Mask PII in logs and traces

### Performance
- [ ] Use batch processors
- [ ] Configure memory limits
- [ ] Implement backpressure handling
- [ ] Monitor collector resource usage

---

**Next Steps**:
- Learn [Monitoring & Observability](./monitoring-observability.md)
- Explore [Service Mesh](./service-mesh.md)
- Master [Kubernetes Operations](./kubernetes-operations.md)


