# Kubernetes Networking

## 🎯 Networking Model

### Pod Networking
- Every pod gets unique IP
- Pods can communicate directly
- No NAT between pods

### Service Types
- **ClusterIP**: Internal only
- **NodePort**: External via node IP
- **LoadBalancer**: External via cloud LB
- **ExternalName**: External service

## 📝 Service Examples

### ClusterIP
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
```

### NodePort
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
    nodePort: 30080
```

### LoadBalancer
```yaml
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
```

## 🔧 Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
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
```

## 🌐 Network Policies

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: app-network-policy
spec:
  podSelector:
    matchLabels:
      app: nginx
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 80
```

## ✅ Best Practices

- Use Services for discovery
- Implement Network Policies
- Use Ingress for HTTP/HTTPS
- Understand CNI plugins
- Monitor network traffic
- Secure network communication

---

**Next**: Learn Kubernetes storage.

