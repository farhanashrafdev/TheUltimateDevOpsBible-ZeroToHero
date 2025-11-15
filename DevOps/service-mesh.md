# Service Mesh

## 🎯 What is Service Mesh?

Service mesh is a dedicated infrastructure layer for managing service-to-service communication, providing observability, security, and traffic management.

## 📚 Core Concepts

### Service Mesh Components
- **Data Plane**: Proxies handling traffic
- **Control Plane**: Management and configuration
- **Sidecar**: Proxy per service instance

### Benefits
- Traffic management
- Security (mTLS)
- Observability
- Policy enforcement

## 🛠️ Istio

### Installation
```bash
istioctl install --set profile=default
```

### VirtualService
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    route:
    - destination:
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v1
```

### DestinationRule
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

## 🔐 Security

### mTLS
```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
spec:
  mtls:
    mode: STRICT
```

## ✅ Best Practices

- Start with basic setup
- Enable mTLS gradually
- Use traffic policies
- Monitor service mesh
- Understand performance impact
- Implement proper observability

---

**Next**: Learn monitoring and observability.

