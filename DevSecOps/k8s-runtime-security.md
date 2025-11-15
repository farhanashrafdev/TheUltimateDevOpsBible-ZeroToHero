# Kubernetes Runtime Security

## 🎯 Securing Kubernetes Clusters

Runtime security for Kubernetes involves protecting pods, clusters, and workloads.

## 🔒 Security Controls

### Pod Security Policies
- Restrict capabilities
- Prevent privilege escalation
- Enforce security contexts

### Network Policies
- Control pod communication
- Micro-segmentation
- Ingress/egress rules

### RBAC
- Role-based access
- Least privilege
- Service accounts

## 🛠️ Tools

### Falco
- Runtime threat detection
- Behavioral analysis
- Policy enforcement

### OPA Gatekeeper
- Policy enforcement
- Admission control
- Custom policies

### Kyverno
- Policy engine
- Resource validation
- Image verification

## 📝 Examples

### Network Policy
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: app-policy
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
```

## ✅ Best Practices

- Enable Pod Security Standards
- Implement network policies
- Use RBAC
- Monitor runtime
- Regular audits
- Update regularly

---

**Next**: Learn supply chain security.

