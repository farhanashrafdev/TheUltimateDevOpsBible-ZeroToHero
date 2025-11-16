# Kubernetes Networking: Complete Guide

## 🎯 Introduction

Kubernetes networking is complex but crucial for understanding how pods communicate, how services work, and how external traffic reaches your applications. This guide covers all aspects of Kubernetes networking.

### Kubernetes Networking Model

**Core Principles**:
1. **Every pod gets its own IP**: No NAT between pods
2. **Pods can communicate directly**: Using pod IPs
3. **Containers in a pod share network**: Same IP, can use localhost
4. **Service abstraction**: Stable IP for pod groups

## 📚 Network Architecture

### Pod Networking

```
┌─────────────────────────────────────────┐
│         Kubernetes Cluster              │
│                                         │
│  ┌──────────┐      ┌──────────┐        │
│  │   Pod 1  │─────│   Pod 2  │        │
│  │ 10.0.1.5 │     │ 10.0.1.6 │        │
│  └──────────┘      └──────────┘        │
│       │                  │             │
│       └──────────┬───────┘             │
│                  │                     │
│            ┌─────▼─────┐               │
│            │  Service  │               │
│            │ 10.0.2.10 │               │
│            └───────────┘               │
│                                         │
└─────────────────────────────────────────┘
```

### CNI (Container Network Interface)

**CNI Plugins**:
- **Calico**: Policy-driven networking
- **Flannel**: Simple overlay network
- **Weave**: Encrypted networking
- **Cilium**: eBPF-based networking
- **Antrea**: VMware's CNI

## 🌐 Services

### Service Types

#### ClusterIP (Default)

**Internal-only service**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: internal-service
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
    name: http
  - port: 443
    targetPort: 8443
    protocol: TCP
    name: https
```

#### NodePort

**External access via node IP**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nodeport-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080  # Optional, auto-assigned if not specified
    protocol: TCP
    name: http
```

**Access**: `http://<node-ip>:30080`

#### LoadBalancer

**External access via cloud load balancer**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: lb-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
  loadBalancerIP: 1.2.3.4  # Optional, cloud-specific
```

#### ExternalName

**External service alias**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  type: ExternalName
  externalName: database.example.com
  ports:
  - port: 5432
```

### Headless Service

**Direct pod access (no load balancing)**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: headless-service
spec:
  clusterIP: None  # Headless
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 8080
```

**Use Cases**:
- StatefulSets
- Service discovery
- Direct pod access

### Service Discovery

**DNS-based discovery**:

```bash
# Service DNS format
<service-name>.<namespace>.svc.cluster.local

# Examples
nginx.default.svc.cluster.local
api.production.svc.cluster.local
```

**In Pod**:
```yaml
env:
- name: API_URL
  value: "http://api-service:8080"
```

## 🚪 Ingress

### Basic Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - app.example.com
    secretName: app-tls
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-service
            port:
              number: 80
```

### Multiple Paths

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multi-path-ingress
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
      - path: /web
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: default-service
            port:
              number: 80
```

### Multiple Hosts

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multi-host-ingress
spec:
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-service
            port:
              number: 80
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
```

### Ingress Controllers

**Popular Controllers**:
- **NGINX Ingress**: Most popular
- **Traefik**: Modern, feature-rich
- **HAProxy**: High performance
- **Istio Gateway**: Service mesh integration
- **AWS ALB Ingress**: AWS-specific

**Install NGINX Ingress**:
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml
```

## 🛡️ Network Policies

### Basic Network Policy

**Deny all traffic**:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: production
spec:
  podSelector: {}  # All pods
  policyTypes:
  - Ingress
  - Egress
```

### Allow Specific Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
```

### Allow Specific Egress

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-egress
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: app
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 5432
  - to:
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - protocol: UDP
      port: 53  # DNS
  - to: []  # Allow all external
    ports:
    - protocol: TCP
      port: 443
    - protocol: TCP
      port: 80
```

### Advanced Network Policy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: advanced-policy
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
      namespaceSelector:
        matchLabels:
          name: production
    - ipBlock:
        cidr: 10.0.0.0/8
        except:
        - 10.0.1.0/24
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 5432
  - to: []
    ports:
    - protocol: TCP
      port: 443
      port: 80
```

## 🔌 DNS

### CoreDNS Configuration

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        health {
           lameduck 5s
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
           ttl 30
        }
        prometheus :9153
        forward . /etc/resolv.conf {
           max_concurrent 1000
        }
        cache 30
        loop
        reload
        loadbalance
    }
```

### DNS Records

**Service DNS**:
- `service-name.namespace.svc.cluster.local`
- `service-name.namespace.svc`
- `service-name.namespace`
- `service-name` (same namespace)

**Pod DNS**:
- `pod-ip.namespace.pod.cluster.local`

## 🌍 External Access Patterns

### Port Forwarding

```bash
# Forward local port to pod
kubectl port-forward pod/pod-name 8080:80

# Forward to service
kubectl port-forward svc/service-name 8080:80

# Forward to deployment
kubectl port-forward deployment/deployment-name 8080:80
```

### NodePort Access

```yaml
# Service with NodePort
apiVersion: v1
kind: Service
metadata:
  name: nodeport-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
```

**Access**: `http://<any-node-ip>:30080`

### LoadBalancer Access

```yaml
# Cloud LoadBalancer
apiVersion: v1
kind: Service
metadata:
  name: lb-service
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
```

## 🔍 Network Troubleshooting

### Common Commands

```bash
# Check service endpoints
kubectl get endpoints service-name

# Describe service
kubectl describe svc service-name

# Test DNS resolution
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup service-name

# Test connectivity
kubectl run -it --rm debug --image=busybox --restart=Never -- wget -O- http://service-name:80

# Check network policies
kubectl get networkpolicies
kubectl describe networkpolicy policy-name

# View iptables rules (on node)
sudo iptables -t nat -L
```

### Debugging Network Issues

```bash
# Pod can't reach service
kubectl exec pod-name -- nslookup service-name
kubectl exec pod-name -- ping service-name
kubectl get endpoints service-name

# Service not accessible externally
kubectl get svc
kubectl describe svc service-name
kubectl get ingress
kubectl describe ingress ingress-name

# Network policy blocking
kubectl get networkpolicies -n namespace
kubectl describe networkpolicy policy-name
```

## ✅ Best Practices

### 1. Use Services for Discovery

```yaml
# Don't use pod IPs directly
# Use service names
```

### 2. Implement Network Policies

```yaml
# Start with deny-all
# Then allow specific traffic
```

### 3. Use Ingress for HTTP/HTTPS

```yaml
# Don't expose services directly
# Use Ingress with TLS
```

### 4. Understand CNI Plugin

```bash
# Know which CNI you're using
# Understand its capabilities
```

### 5. Monitor Network Traffic

```yaml
# Use network monitoring tools
# Track bandwidth usage
```

### 6. Secure Network Communication

```yaml
# Use TLS/SSL
# Implement network policies
# Use service mesh for advanced features
```

## ✅ Mastery Checklist

- [ ] Understand pod networking
- [ ] Configure different service types
- [ ] Set up Ingress with TLS
- [ ] Implement network policies
- [ ] Troubleshoot network issues
- [ ] Use DNS for service discovery
- [ ] Configure external access
- [ ] Understand CNI plugins
- [ ] Monitor network traffic
- [ ] Secure network communication

---

**Next Steps**:
- Learn [Service Mesh](./service-mesh.md) for advanced networking
- Explore [Kubernetes Storage](./kubernetes-storage.md)
- Master [Kubernetes Operations](./kubernetes-operations.md)

**Remember**: Kubernetes networking is complex but powerful. Start with services and ingress, then add network policies for security. Always test connectivity and monitor network traffic.
