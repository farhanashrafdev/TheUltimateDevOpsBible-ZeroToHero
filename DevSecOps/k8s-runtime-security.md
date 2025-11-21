# Kubernetes Runtime Security: Protecting Running Workloads

## 🎯 Introduction

Container images are scanned before deployment, but what about runtime threats? Runtime security detects and prevents malicious behavior in running containers.

## 🛡️ Runtime Security Tools

### 1. Falco - Runtime Threat Detection

**What it does**: Monitors system calls to detect anomalous behavior

**Installation**:
```bash
# Helm
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm install falco falcosecurity/falco

# Verify
kubectl get pods -n falco
```

**Example Rules**:
```yaml
# Detect shell spawned in container
- rule: Terminal shell in container
  desc: A shell was spawned in a container
  condition: >
    spawned_process and container
    and shell_procs
  output: >
    Shell spawned in container
    (user=%user.name container=%container.name
    shell=%proc.name)
  priority: WARNING

# Detect sensitive file access
- rule: Read sensitive file
  desc: Attempt to read sensitive files
  condition: >
    open_read and container
    and fd.name in (sensitive_files)
  output: >
    Sensitive file opened for reading
    (user=%user.name file=%fd.name)
  priority: WARNING
```

### 2. Tetragon - eBPF-based Security

**What it does**: Uses eBPF for deep kernel-level visibility

```bash
# Install Tetragon
helm install tetragon cilium/tetragon -n kube-system

# Monitor events
kubectl logs -n kube-system -l app.kubernetes.io/name=tetragon -c export-stdout -f
```

---

## 🔒 Pod Security Standards

### Three Levels

1. **Privileged**: Unrestricted (not recommended)
2. **Baseline**: Minimal restrictions
3. **Restricted**: Heavily restricted (recommended)

### Enforce with Labels

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

### Restricted Pod Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: myapp:latest
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        drop:
          - ALL
      readOnlyRootFilesystem: true
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
```

---

## 🚫 Network Policies

Restrict pod-to-pod communication.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-policy
  namespace: production
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
```

---

## 🔐 Admission Controllers

### OPA Gatekeeper

**Install**:
```bash
kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/master/deploy/gatekeeper.yaml
```

**Policy: Require Labels**:
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
          msg := sprintf("Missing required labels: %v", [missing])
        }
```

**Apply Constraint**:
```yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: require-labels
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
  parameters:
    labels: ["app", "environment"]
```

---

## 🎯 Complete Runtime Security Stack

```yaml
# 1. Install Falco
helm install falco falcosecurity/falco \
  --set falco.grpc.enabled=true \
  --set falco.grpcOutput.enabled=true

# 2. Configure Pod Security Standards
kubectl label namespace production \
  pod-security.kubernetes.io/enforce=restricted

# 3. Apply Network Policies
kubectl apply -f network-policies/

# 4. Install OPA Gatekeeper
kubectl apply -f gatekeeper.yaml

# 5. Apply Constraints
kubectl apply -f constraints/
```

---

## ✅ Best Practices

- [ ] Deploy Falco or Tetragon
- [ ] Enforce Pod Security Standards (restricted)
- [ ] Implement Network Policies
- [ ] Use admission controllers (OPA/Kyverno)
- [ ] Run containers as non-root
- [ ] Drop all capabilities
- [ ] Use read-only root filesystem
- [ ] Set resource limits
- [ ] Enable seccomp profiles
- [ ] Monitor runtime events

---

## 📚 Tools

- **Falco**: Runtime threat detection
- **Tetragon**: eBPF-based security
- **OPA Gatekeeper**: Policy enforcement
- **Kyverno**: Kubernetes-native policies
- **Tracee**: Runtime security and forensics

---

**Next**: Learn about [Zero Trust](./zero-trust.md) and [IaC Scanning](./iac-scanning.md).
