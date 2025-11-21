# Zero Trust Architecture: Never Trust, Always Verify

## 🎯 Introduction

Traditional security: Trust everything inside the network perimeter.
Zero Trust: **Never trust, always verify** - even inside your network.

## 🔐 Core Principles

1. **Verify Explicitly**: Always authenticate and authorize
2. **Least Privilege**: Minimum necessary access
3. **Assume Breach**: Design as if already compromised

---

## 🏗️ Zero Trust Components

### 1. Identity-Based Access

**No more network-based trust**:
```
Traditional: "You're on the corporate network? Come in!"
Zero Trust: "Who are you? Prove it. What do you need? Why?"
```

**Implementation**:
- mTLS for service-to-service
- OIDC/OAuth for user authentication
- Service mesh for identity

### 2. Micro-Segmentation

**Network Policies**:
```yaml
# Only allow specific pod-to-pod communication
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: zero-trust-policy
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
  egress:
    - to:
        - podSelector:
            matchLabels:
              app: database
```

### 3. Continuous Verification

**Not just login, but every request**:
```
User logs in → Token issued (short-lived)
Every API call → Verify token
Token expires → Re-authenticate
```

---

## 🕸️ Service Mesh for Zero Trust

### Istio mTLS

**Automatic mTLS between services**:
```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: production
spec:
  mtls:
    mode: STRICT  # Require mTLS for all traffic
```

**Authorization Policy**:
```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: api-authz
  namespace: production
spec:
  selector:
    matchLabels:
      app: api
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

---

## 🔑 Identity and Access Management

### Workload Identity

**AWS**:
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789:role/MyAppRole
```

**GCP**:
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app
  annotations:
    iam.gke.io/gcp-service-account: my-app@project.iam.gserviceaccount.com
```

---

## 🎯 Complete Zero Trust Implementation

### 1. Service Mesh Setup

```bash
# Install Istio
istioctl install --set profile=default

# Enable sidecar injection
kubectl label namespace production istio-injection=enabled

# Enforce mTLS
kubectl apply -f - <<EOF
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: istio-system
spec:
  mtls:
    mode: STRICT
EOF
```

### 2. Network Policies

```bash
# Deny all by default
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: production
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
EOF

# Allow specific traffic
kubectl apply -f network-policies/
```

### 3. RBAC

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: production
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: production
subjects:
  - kind: ServiceAccount
    name: my-app
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

---

## ✅ Best Practices

- [ ] Implement mTLS everywhere
- [ ] Use short-lived credentials
- [ ] Enforce least privilege
- [ ] Implement micro-segmentation
- [ ] Continuous authentication
- [ ] Monitor all access
- [ ] Assume breach
- [ ] Encrypt data in transit and at rest

---

## 📚 Tools

- **Service Mesh**: Istio, Linkerd
- **Identity**: SPIFFE/SPIRE
- **Network**: Calico, Cilium
- **Secrets**: Vault, External Secrets

---

**Next**: Learn about [IaC Scanning](./iac-scanning.md) and [Secrets Management](./secrets-management.md).
