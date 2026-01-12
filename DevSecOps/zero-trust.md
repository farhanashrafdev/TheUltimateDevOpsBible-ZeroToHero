# Zero Trust Architecture

## 🎯 Introduction

Zero Trust is a security model based on "never trust, always verify" - treating every access request as potentially hostile regardless of origin.

## 📚 Core Principles

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Zero Trust Principles                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Verify Explicitly                                                │
│     └── Authenticate and authorize every request                    │
│                                                                      │
│  2. Least Privilege Access                                           │
│     └── Just-enough-access, just-in-time permissions                │
│                                                                      │
│  3. Assume Breach                                                    │
│     └── Minimize blast radius, segment access, encrypt everything   │
│                                                                      │
│  Traditional Model:              Zero Trust Model:                   │
│  ┌──────────────┐               ┌──────────────┐                    │
│  │   Trusted    │               │  Untrusted   │                    │
│  │   Network    │               │   Network    │                    │
│  │  (Castle)    │               │ (Verify All) │                    │
│  └──────────────┘               └──────────────┘                    │
│         │                              │                             │
│         ▼                              ▼                             │
│  ┌──────────────┐               Each request verified:               │
│  │  Firewall    │               • Identity                          │
│  │   (Moat)     │               • Device                            │
│  └──────────────┘               • Context                           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 🔐 Identity-Based Access

### Service Mesh (Istio)

```yaml
# PeerAuthentication - require mTLS
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: production
spec:
  mtls:
    mode: STRICT

---
# AuthorizationPolicy - allow specific services
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: api-access
  namespace: production
spec:
  selector:
    matchLabels:
      app: api
  action: ALLOW
  rules:
    - from:
        - source:
            principals:
              - "cluster.local/ns/production/sa/frontend"
      to:
        - operation:
            methods: ["GET", "POST"]
            paths: ["/api/v1/*"]
```

### SPIFFE/SPIRE

```yaml
# ClusterSPIFFEID for workload identity
apiVersion: spire.spiffe.io/v1alpha1
kind: ClusterSPIFFEID
metadata:
  name: app-identity
spec:
  spiffeIDTemplate: "spiffe://company.com/ns/{{ .PodMeta.Namespace }}/sa/{{ .PodSpec.ServiceAccountName }}"
  podSelector:
    matchLabels:
      spiffe: enabled
```

## 🌐 Network Segmentation

### Kubernetes Network Policies

```yaml
# Default deny all
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress

---
# Allow only verified traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-verified
spec:
  podSelector:
    matchLabels:
      app: api
  ingress:
    - from:
        - podSelector:
            matchLabels:
              verified: "true"
```

### Cilium Identity-Aware Policies

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: api-policy
spec:
  endpointSelector:
    matchLabels:
      app: api
  ingress:
    - fromEndpoints:
        - matchLabels:
            app: frontend
      toPorts:
        - ports:
            - port: "8080"
          rules:
            http:
              - method: "GET"
                path: "/api/.*"
                headers:
                  - 'X-Auth-Token: ^[a-zA-Z0-9]+$'
```

## 🔑 Just-In-Time Access

### Teleport for SSH/K8s Access

```yaml
# Teleport role with request workflow
kind: role
metadata:
  name: developer
spec:
  allow:
    logins: ['{{internal.logins}}']
    kubernetes_groups: ['developers']
    request:
      roles: ['admin']
      max_duration: 1h
  options:
    require_session_mfa: true
```

### AWS IAM Identity Center

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sts:AssumeRole",
      "Resource": "arn:aws:iam::*:role/ProductionAccess",
      "Condition": {
        "Bool": {"aws:MultiFactorAuthPresent": "true"},
        "IpAddress": {"aws:SourceIp": "10.0.0.0/8"}
      }
    }
  ]
}
```

## 📊 Continuous Verification

```yaml
# OPA policy for continuous auth
package kubernetes.admission

deny[msg] {
  input.request.kind.kind == "Pod"
  
  # Verify service account has valid identity
  sa := input.request.object.spec.serviceAccountName
  not valid_service_account(sa)
  
  msg := sprintf("Service account %v not authorized", [sa])
}

valid_service_account(sa) {
  allowed := ["app", "frontend", "api"]
  sa == allowed[_]
}

# Verify all traffic is authenticated
deny[msg] {
  input.request.kind.kind == "Service"
  not has_mtls_annotation(input.request.object)
  msg := "Service must have mTLS enabled"
}
```

## ✅ Zero Trust Checklist

- [ ] Identity for all workloads (SPIFFE/mTLS)
- [ ] Default deny network policies
- [ ] Just-in-time privileged access
- [ ] Continuous verification (not just at login)
- [ ] Micro-segmentation
- [ ] Encrypted all data (transit + rest)
- [ ] Comprehensive logging and monitoring
- [ ] Device posture verification

---

**Next**: Return to [DevSecOps Overview](./overview.md).

