# Kubernetes Runtime Security

## 🎯 Introduction

Runtime security protects Kubernetes clusters during execution, detecting and preventing threats that static analysis cannot catch.

## 📚 Security Layers

```
┌─────────────────────────────────────────────────────────────────────┐
│                 Kubernetes Runtime Security Stack                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Layer 4: Threat Detection                                           │
│  └── Falco, Tetragon, Sysdig                                        │
│                                                                      │
│  Layer 3: Network Policies                                           │
│  └── Calico, Cilium, NetworkPolicy                                  │
│                                                                      │
│  Layer 2: Pod Security                                               │
│  └── Pod Security Standards, SecurityContext                        │
│                                                                      │
│  Layer 1: RBAC & Admission Control                                   │
│  └── OPA/Gatekeeper, Kyverno, PSA                                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 🛡️ Pod Security Standards (PSS)

### Namespace Configuration

```yaml
# Enforce restricted security standard
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/enforce-version: latest
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

### Compliant Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-app
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 1000
    fsGroup: 1000
    seccompProfile:
      type: RuntimeDefault
  containers:
    - name: app
      image: myapp:v1.0
      securityContext:
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
        capabilities:
          drop:
            - ALL
      resources:
        limits:
          memory: "256Mi"
          cpu: "500m"
        requests:
          memory: "128Mi"
          cpu: "250m"
```

## 🔍 Falco (Runtime Threat Detection)

### Installation

```bash
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm install falco falcosecurity/falco \
  --set falcosidekick.enabled=true \
  --set falcosidekick.webui.enabled=true
```

### Custom Rules

```yaml
# custom-rules.yaml
- rule: Shell in Container
  desc: Detect shell execution in container
  condition: >
    spawned_process and
    container and
    proc.name in (bash, sh, zsh, dash)
  output: >
    Shell spawned in container (user=%user.name container=%container.name
    image=%container.image.repository command=%proc.cmdline)
  priority: WARNING

- rule: Write to /etc
  desc: Detect writes to /etc directory
  condition: >
    open_write and
    container and
    fd.name startswith /etc
  output: >
    File write to /etc (user=%user.name container=%container.name
    file=%fd.name)
  priority: ERROR

- rule: Crypto Mining
  desc: Detect crypto mining processes
  condition: >
    spawned_process and
    container and
    (proc.name in (xmrig, minerd) or
     proc.cmdline contains "stratum+tcp")
  output: >
    Crypto mining detected (container=%container.name proc=%proc.name)
  priority: CRITICAL
```

### Falco with Slack Alerts

```yaml
# falcosidekick values
config:
  slack:
    webhookurl: "https://hooks.slack.com/services/XXX"
    minimumpriority: warning
    outputformat: "all"
```

## 🌐 Network Policies

### Default Deny

```yaml
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
```

### Allow Specific Traffic

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-app-traffic
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: web
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
          port: 8080
  egress:
    - to:
        - podSelector:
            matchLabels:
              app: database
      ports:
        - protocol: TCP
          port: 5432
    - to:
        - namespaceSelector: {}
          podSelector:
            matchLabels:
              k8s-app: kube-dns
      ports:
        - protocol: UDP
          port: 53
```

## 🔒 OPA Gatekeeper

### Constraint Template

```yaml
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlabels
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLabels
      validation:
        openAPIV3Schema:
          type: object
          properties:
            labels:
              type: array
              items:
                type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredlabels
        
        violation[{"msg": msg}] {
          provided := {label | input.review.object.metadata.labels[label]}
          required := {label | label := input.parameters.labels[_]}
          missing := required - provided
          count(missing) > 0
          msg := sprintf("Missing labels: %v", [missing])
        }
```

### Constraint

```yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: require-team-label
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
  parameters:
    labels:
      - "team"
      - "app"
```

## 🐝 Cilium (eBPF-based Security)

### Network Policy with L7 Filtering

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: http-policy
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
              protocol: TCP
          rules:
            http:
              - method: "GET"
                path: "/api/v1/.*"
              - method: "POST"
                path: "/api/v1/users"
```

## 📊 Runtime Monitoring Pipeline

```yaml
# Complete monitoring setup
apiVersion: v1
kind: ConfigMap
metadata:
  name: falco-config
data:
  falco.yaml: |
    json_output: true
    json_include_output_property: true
    http_output:
      enabled: true
      url: "http://falco-exporter:2801/"
---
# Prometheus metrics from Falco
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: falco
spec:
  selector:
    matchLabels:
      app: falco
  endpoints:
    - port: metrics
```

## ✅ Security Checklist

- [ ] Pod Security Standards enforced
- [ ] Falco installed with custom rules
- [ ] Network policies (default deny + allow)
- [ ] RBAC properly configured
- [ ] Admission controllers (OPA/Kyverno)
- [ ] Secrets encrypted at rest
- [ ] Audit logging enabled
- [ ] Container images signed and verified

---

**Next**: Learn about [Supply Chain Security](./supply-chain-security.md).

