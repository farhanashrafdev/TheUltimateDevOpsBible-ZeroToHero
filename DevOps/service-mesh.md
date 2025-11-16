# Service Mesh: Complete Guide to Microservices Networking

## 🎯 Introduction

Service mesh is a dedicated infrastructure layer for managing service-to-service communication in microservices architectures. It provides traffic management, security, and observability without requiring changes to application code.

### What is a Service Mesh?

**Service Mesh** handles:
- **Traffic Management**: Routing, load balancing, retries
- **Security**: mTLS, authentication, authorization
- **Observability**: Metrics, logs, traces
- **Policy Enforcement**: Rate limiting, access control

### Service Mesh Architecture

```
┌─────────────────────────────────────────┐
│         Application Pod                  │
│  ┌──────────┐      ┌──────────┐         │
│  │   App    │─────│  Sidecar │         │
│  │ Container│     │  Proxy   │         │
│  └──────────┘      └──────────┘         │
└─────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────┐
│         Control Plane                    │
│  • Configuration                         │
│  • Policy Management                     │
│  • Certificate Management                │
└─────────────────────────────────────────┘
```

## 🛠️ Istio

**Most popular service mesh**

### Installation

```bash
# Download Istio
curl -L https://istio.io/downloadIstio | sh -
cd istio-*

# Install with default profile
istioctl install --set profile=default

# Verify installation
istioctl verify-install
kubectl get pods -n istio-system
```

### Enable Sidecar Injection

```bash
# Enable for namespace
kubectl label namespace production istio-injection=enabled

# Or use automatic injection
kubectl label namespace production istio-injection=enabled --overwrite
```

### VirtualService

**Traffic routing rules**

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
  namespace: production
spec:
  hosts:
  - reviews
  http:
  # Route to v2 for specific user
  - match:
    - headers:
        end-user:
          exact: jason
    route:
    - destination:
        host: reviews
        subset: v2
      weight: 100
  
  # Route to v1 for others
  - route:
    - destination:
        host: reviews
        subset: v1
      weight: 100
```

### DestinationRule

**Subset definitions and policies**

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: reviews
  namespace: production
spec:
  host: reviews
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
    trafficPolicy:
      loadBalancer:
        simple: LEAST_CONN
      connectionPool:
        tcp:
          maxConnections: 100
        http:
          http1MaxPendingRequests: 10
          http2MaxRequests: 100
          maxRequestsPerConnection: 2
      outlierDetection:
        consecutiveErrors: 3
        interval: 30s
        baseEjectionTime: 30s
        maxEjectionPercent: 50
```

### Canary Deployment

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: canary-deployment
spec:
  hosts:
  - myapp
  http:
  - match:
    - headers:
        canary:
          exact: "true"
    route:
    - destination:
        host: myapp
        subset: canary
      weight: 100
  - route:
    - destination:
        host: myapp
        subset: stable
      weight: 90
    - destination:
        host: myapp
        subset: canary
      weight: 10
```

### Circuit Breaker

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: circuit-breaker
spec:
  host: myapp
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 10
        http2MaxRequests: 100
        maxRequestsPerConnection: 2
        maxRetries: 3
    outlierDetection:
      consecutiveErrors: 5
      interval: 30s
      baseEjectionTime: 30s
      maxEjectionPercent: 50
      minHealthPercent: 50
```

### Retry Policy

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: retry-policy
spec:
  hosts:
  - myapp
  http:
  - match:
    - uri:
        prefix: /api
    route:
    - destination:
        host: myapp
    retries:
      attempts: 3
      perTryTimeout: 2s
      retryOn: 5xx,reset,connect-failure,refused-stream
```

### Timeout

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: timeout-policy
spec:
  hosts:
  - myapp
  http:
  - timeout: 10s
    route:
    - destination:
        host: myapp
```

## 🔐 Security

### mTLS (Mutual TLS)

**Encrypt all service-to-service traffic**

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: production
spec:
  mtls:
    mode: STRICT
```

**Modes**:
- **STRICT**: mTLS required
- **PERMISSIVE**: mTLS optional
- **DISABLE**: No mTLS

### Authorization Policy

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: production
spec:
  selector:
    matchLabels:
      app: backend
  action: ALLOW
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/production/sa/frontend"]
    to:
    - operation:
        methods: ["GET", "POST"]
        paths: ["/api/*"]
```

### JWT Authentication

```yaml
apiVersion: security.istio.io/v1beta1
kind: RequestAuthentication
metadata:
  name: jwt-auth
  namespace: production
spec:
  selector:
    matchLabels:
      app: api
  jwtRules:
  - issuer: "https://auth.example.com"
    jwksUri: "https://auth.example.com/.well-known/jwks.json"
```

## 📊 Observability

### Metrics

**Automatic metrics collection**:
- Request count
- Request duration
- Error rate
- TCP metrics

### Access Logs

```yaml
apiVersion: telemetry.istio.io/v1alpha1
kind: Telemetry
metadata:
  name: access-logging
  namespace: production
spec:
  accessLogging:
  - providers:
    - name: envoy
```

### Distributed Tracing

```yaml
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  name: control-plane
spec:
  meshConfig:
    defaultProviders:
      tracing:
      - zipkin
    extensionProviders:
    - name: zipkin
      zipkin:
        service: zipkin.istio-system.svc.cluster.local
        port: 9411
```

## 🚦 Traffic Management

### Load Balancing

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: load-balancing
spec:
  host: myapp
  trafficPolicy:
    loadBalancer:
      simple: LEAST_CONN  # or ROUND_ROBIN, RANDOM, PASSTHROUGH
```

### Fault Injection

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: fault-injection
spec:
  hosts:
  - myapp
  http:
  - fault:
      delay:
        percentage:
          value: 50
        fixedDelay: 5s
      abort:
        percentage:
          value: 10
        httpStatus: 500
    route:
    - destination:
        host: myapp
```

### Mirroring

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: mirror-traffic
spec:
  hosts:
  - myapp
  http:
  - route:
    - destination:
        host: myapp
        subset: v1
      weight: 100
    mirror:
      host: myapp
      subset: v2
    mirrorPercentage:
      value: 10
```

## 🔧 Other Service Meshes

### Linkerd

**Lightweight, simple service mesh**

```bash
# Install Linkerd
curl --proto '=https' --tlsv1.2 -sSfL https://run.linkerd.io/install | sh
linkerd install | kubectl apply -f -
linkerd check
```

### Consul Connect

**HashiCorp's service mesh**

```bash
# Install Consul
helm install consul hashicorp/consul --set connectInject.enabled=true
```

### Kuma

**Universal service mesh**

```bash
# Install Kuma
kumactl install control-plane | kubectl apply -f -
```

## ✅ Best Practices

### 1. Start Simple

```yaml
# Begin with basic setup
# Enable mTLS gradually
# Add policies incrementally
```

### 2. Use Traffic Policies

```yaml
# Implement retries
# Set timeouts
# Use circuit breakers
```

### 3. Monitor Service Mesh

```yaml
# Track mesh metrics
# Monitor sidecar health
# Watch resource usage
```

### 4. Understand Performance Impact

```yaml
# Sidecars add latency
# Monitor overhead
# Optimize configuration
```

### 5. Implement Proper Observability

```yaml
# Use mesh metrics
# Enable distributed tracing
# Set up dashboards
```

## ✅ Mastery Checklist

- [ ] Install and configure service mesh
- [ ] Enable mTLS
- [ ] Configure traffic routing
- [ ] Implement circuit breakers
- [ ] Set up retry policies
- [ ] Use authorization policies
- [ ] Monitor service mesh
- [ ] Implement canary deployments
- [ ] Troubleshoot mesh issues
- [ ] Optimize performance

---

**Next Steps**:
- Learn [Kubernetes Networking](./kubernetes-networking.md)
- Explore [Monitoring & Observability](./monitoring-observability.md)
- Master [Production Practices](./production-practices.md)

**Remember**: Service mesh adds powerful capabilities but also complexity. Start simple, enable features gradually, and always monitor the impact. Use service mesh for advanced traffic management and security, not as a replacement for good application design.
