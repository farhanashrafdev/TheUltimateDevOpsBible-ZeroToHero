# Flux CD: GitOps for Kubernetes

## 🎯 Introduction

Flux is a set of continuous delivery tools for Kubernetes that keeps clusters in sync with configuration sources like Git repositories. It automates deployments based on the GitOps principle: Git as the single source of truth.

### Flux vs Argo CD

```yaml
┌─────────────────┬──────────────────────┬──────────────────────┐
│ Feature         │ Flux                 │ Argo CD              │
├─────────────────┼──────────────────────┼──────────────────────┤
│ Architecture    │ Controllers (native) │ Server + Controller  │
│ UI              │ Weave GitOps (addon) │ Built-in             │
│ Multi-tenancy   │ Native               │ Projects/RBAC        │
│ Helm Support    │ Native controller    │ Native               │
│ Kustomize       │ Native controller    │ Native               │
│ Image Automation│ Built-in             │ Image Updater addon  │
│ Notifications   │ Built-in             │ Built-in             │
│ Progressive     │ Flagger addon        │ Rollouts addon       │
│ CNCF Status     │ Graduated            │ Graduated            │
└─────────────────┴──────────────────────┴──────────────────────┘
```

## 🏗️ Flux Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      Flux Architecture                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────┐                                            │
│  │   Git Repository│                                            │
│  │  (Source of     │                                            │
│  │   Truth)        │                                            │
│  └────────┬────────┘                                            │
│           │ watch & pull                                         │
│           ▼                                                      │
│  ┌────────────────────────────────────────────────────┐         │
│  │               Flux Controllers                      │         │
│  │  ┌──────────────┐  ┌──────────────┐               │         │
│  │  │Source        │  │Kustomize     │               │         │
│  │  │Controller    │  │Controller    │               │         │
│  │  │              │  │              │               │         │
│  │  │ • Git        │  │ • Kustomize  │               │         │
│  │  │ • Helm Repos │  │ • Overlays   │               │         │
│  │  │ • OCI        │  │ • Patches    │               │         │
│  │  └──────────────┘  └──────────────┘               │         │
│  │                                                    │         │
│  │  ┌──────────────┐  ┌──────────────┐               │         │
│  │  │Helm          │  │Notification  │               │         │
│  │  │Controller    │  │Controller    │               │         │
│  │  │              │  │              │               │         │
│  │  │ • Charts     │  │ • Slack      │               │         │
│  │  │ • Values     │  │ • Teams      │               │         │
│  │  │ • Releases   │  │ • Webhooks   │               │         │
│  │  └──────────────┘  └──────────────┘               │         │
│  │                                                    │         │
│  │  ┌──────────────┐                                 │         │
│  │  │Image         │                                 │         │
│  │  │Automation    │                                 │         │
│  │  │              │                                 │         │
│  │  │ • Scanning   │                                 │         │
│  │  │ • Updating   │                                 │         │
│  │  └──────────────┘                                 │         │
│  └────────────────────────────────────────────────────┘         │
│           │ reconcile                                            │
│           ▼                                                      │
│  ┌─────────────────┐                                            │
│  │ Kubernetes      │                                            │
│  │ Cluster         │                                            │
│  └─────────────────┘                                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## 📦 Installation

### Bootstrap with GitHub

```bash
# Install Flux CLI
curl -s https://fluxcd.io/install.sh | sudo bash

# Verify installation
flux check --pre

# Export GitHub token
export GITHUB_TOKEN=<your-token>

# Bootstrap Flux
flux bootstrap github \
  --owner=my-org \
  --repository=fleet-infra \
  --branch=main \
  --path=clusters/production \
  --personal

# For private repo
flux bootstrap github \
  --owner=my-org \
  --repository=fleet-infra \
  --branch=main \
  --path=clusters/production \
  --private=true \
  --personal
```

### Bootstrap with GitLab

```bash
export GITLAB_TOKEN=<your-token>

flux bootstrap gitlab \
  --owner=my-group \
  --repository=fleet-infra \
  --branch=main \
  --path=clusters/production \
  --token-auth
```

### Manual Installation

```bash
# Install Flux components
flux install \
  --namespace=flux-system \
  --network-policy=true \
  --components-extra=image-reflector-controller,image-automation-controller
```

## 📂 Repository Structure

```
fleet-infra/
├── clusters/
│   ├── production/
│   │   ├── flux-system/           # Flux components (auto-generated)
│   │   │   ├── gotk-components.yaml
│   │   │   ├── gotk-sync.yaml
│   │   │   └── kustomization.yaml
│   │   ├── infrastructure.yaml    # Infrastructure sources
│   │   └── apps.yaml              # Application sources
│   │
│   └── staging/
│       ├── flux-system/
│       ├── infrastructure.yaml
│       └── apps.yaml
│
├── infrastructure/
│   ├── controllers/               # Cluster-wide controllers
│   │   ├── cert-manager/
│   │   ├── ingress-nginx/
│   │   └── kustomization.yaml
│   │
│   ├── configs/                   # Cluster configs
│   │   ├── namespaces/
│   │   ├── network-policies/
│   │   └── kustomization.yaml
│   │
│   └── sources/                   # Helm repositories
│       ├── bitnami.yaml
│       ├── jetstack.yaml
│       └── kustomization.yaml
│
└── apps/
    ├── base/                      # Base app definitions
    │   ├── backend/
    │   │   ├── deployment.yaml
    │   │   ├── service.yaml
    │   │   └── kustomization.yaml
    │   └── frontend/
    │       ├── deployment.yaml
    │       ├── service.yaml
    │       └── kustomization.yaml
    │
    ├── production/                # Production overlays
    │   ├── backend/
    │   │   └── kustomization.yaml
    │   └── frontend/
    │       └── kustomization.yaml
    │
    └── staging/                   # Staging overlays
        ├── backend/
        │   └── kustomization.yaml
        └── frontend/
            └── kustomization.yaml
```

## 🔧 Core Resources

### GitRepository Source

```yaml
# Git repository source
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: fleet-infra
  namespace: flux-system
spec:
  interval: 1m
  url: https://github.com/my-org/fleet-infra
  branch: main
  secretRef:
    name: flux-system  # SSH key or token
  ignore: |
    # Exclude files
    *.md
    .github/
---
# With SSH key authentication
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: private-repo
  namespace: flux-system
spec:
  interval: 5m
  url: ssh://git@github.com/my-org/private-repo.git
  branch: main
  secretRef:
    name: private-repo-ssh
---
# SSH key secret
apiVersion: v1
kind: Secret
metadata:
  name: private-repo-ssh
  namespace: flux-system
type: Opaque
stringData:
  identity: |
    -----BEGIN OPENSSH PRIVATE KEY-----
    ...
    -----END OPENSSH PRIVATE KEY-----
  known_hosts: |
    github.com ssh-ed25519 AAAAC3...
```

### Kustomization

```yaml
# Kustomization for apps
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: apps
  namespace: flux-system
spec:
  interval: 10m
  retryInterval: 2m
  timeout: 5m
  sourceRef:
    kind: GitRepository
    name: fleet-infra
  path: ./apps/production
  prune: true                    # Remove deleted resources
  wait: true                     # Wait for resources to be ready
  
  # Health checks
  healthChecks:
    - apiVersion: apps/v1
      kind: Deployment
      name: backend
      namespace: production
    - apiVersion: apps/v1
      kind: Deployment
      name: frontend
      namespace: production
  
  # Dependencies
  dependsOn:
    - name: infrastructure
  
  # Patches
  patches:
    - patch: |
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: not-important
        spec:
          replicas: 3
      target:
        kind: Deployment
        labelSelector: "env=production"
  
  # Variable substitution
  postBuild:
    substitute:
      CLUSTER_NAME: production
      REGION: us-east-1
    substituteFrom:
      - kind: ConfigMap
        name: cluster-settings
      - kind: Secret
        name: cluster-secrets
```

### HelmRepository & HelmRelease

```yaml
# Helm repository
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: HelmRepository
metadata:
  name: bitnami
  namespace: flux-system
spec:
  interval: 30m
  url: https://charts.bitnami.com/bitnami
---
# OCI Helm repository
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: HelmRepository
metadata:
  name: podinfo
  namespace: flux-system
spec:
  type: oci
  interval: 5m
  url: oci://ghcr.io/stefanprodan/charts
---
# Helm release
apiVersion: helm.toolkit.fluxcd.io/v2beta2
kind: HelmRelease
metadata:
  name: redis
  namespace: production
spec:
  interval: 30m
  chart:
    spec:
      chart: redis
      version: "18.x"
      sourceRef:
        kind: HelmRepository
        name: bitnami
        namespace: flux-system
  
  install:
    remediation:
      retries: 3
  
  upgrade:
    remediation:
      retries: 3
      remediateLastFailure: true
    cleanupOnFail: true
  
  # Values
  values:
    architecture: replication
    auth:
      enabled: true
    replica:
      replicaCount: 3
  
  # Values from ConfigMap/Secret
  valuesFrom:
    - kind: ConfigMap
      name: redis-values
      valuesKey: values.yaml
    - kind: Secret
      name: redis-secrets
      valuesKey: auth.yaml
  
  # Post-renderers for Kustomize patches
  postRenderers:
    - kustomize:
        patches:
          - target:
              kind: StatefulSet
              name: redis-master
            patch: |
              - op: add
                path: /spec/template/metadata/annotations/prometheus.io~1scrape
                value: "true"
```

## 🔄 Image Automation

### Image Scanning & Updating

```yaml
# Image repository to scan
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImageRepository
metadata:
  name: backend
  namespace: flux-system
spec:
  image: ghcr.io/my-org/backend
  interval: 5m
  secretRef:
    name: ghcr-auth  # Registry credentials
---
# Image policy - which tags to track
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImagePolicy
metadata:
  name: backend
  namespace: flux-system
spec:
  imageRepositoryRef:
    name: backend
  policy:
    semver:
      range: 1.x.x
---
# Alternative: filter by tag pattern
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImagePolicy
metadata:
  name: backend-sha
  namespace: flux-system
spec:
  imageRepositoryRef:
    name: backend
  filterTags:
    pattern: '^main-[a-f0-9]+-(?P<ts>[0-9]+)'
    extract: '$ts'
  policy:
    numerical:
      order: asc
---
# Image update automation
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImageUpdateAutomation
metadata:
  name: flux-system
  namespace: flux-system
spec:
  interval: 30m
  sourceRef:
    kind: GitRepository
    name: fleet-infra
  git:
    checkout:
      ref:
        branch: main
    commit:
      author:
        name: fluxcdbot
        email: fluxcdbot@users.noreply.github.com
      messageTemplate: |
        Automated image update
        
        Automation: {{ .AutomationObject }}
        
        Files:
        {{ range $filename, $_ := .Changed.FileChanges -}}
        - {{ $filename }}
        {{ end -}}
        
        Objects:
        {{ range $resource, $changes := .Changed.Objects -}}
        - {{ $resource.Kind }} {{ $resource.Name }}
          Changes:
        {{ range $_, $change := $changes -}}
            - {{ $change.OldValue }} -> {{ $change.NewValue }}
        {{ end -}}
        {{ end -}}
    push:
      branch: main
  update:
    path: ./apps
    strategy: Setters
```

### Marking Images for Update

```yaml
# In deployment.yaml - mark image for automation
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: production
spec:
  template:
    spec:
      containers:
      - name: backend
        image: ghcr.io/my-org/backend:1.0.0 # {"$imagepolicy": "flux-system:backend"}
```

## 🔔 Notifications

### Slack Notifications

```yaml
# Notification provider
apiVersion: notification.toolkit.fluxcd.io/v1beta2
kind: Provider
metadata:
  name: slack
  namespace: flux-system
spec:
  type: slack
  channel: deployments
  secretRef:
    name: slack-webhook
---
# Webhook secret
apiVersion: v1
kind: Secret
metadata:
  name: slack-webhook
  namespace: flux-system
stringData:
  address: https://hooks.slack.com/services/T00/B00/XXX
---
# Alert configuration
apiVersion: notification.toolkit.fluxcd.io/v1beta2
kind: Alert
metadata:
  name: on-call-alerts
  namespace: flux-system
spec:
  providerRef:
    name: slack
  eventSeverity: error
  eventSources:
    - kind: Kustomization
      name: '*'
    - kind: HelmRelease
      name: '*'
  summary: "Flux alert for production cluster"
---
# All events alert
apiVersion: notification.toolkit.fluxcd.io/v1beta2
kind: Alert
metadata:
  name: all-events
  namespace: flux-system
spec:
  providerRef:
    name: slack
  eventSeverity: info
  eventSources:
    - kind: Kustomization
      name: apps
      namespace: flux-system
  inclusionList:
    - ".*succeeded.*"
```

### GitHub Commit Status

```yaml
apiVersion: notification.toolkit.fluxcd.io/v1beta2
kind: Provider
metadata:
  name: github-status
  namespace: flux-system
spec:
  type: github
  address: https://github.com/my-org/fleet-infra
  secretRef:
    name: github-token
---
apiVersion: notification.toolkit.fluxcd.io/v1beta2
kind: Alert
metadata:
  name: github-status
  namespace: flux-system
spec:
  providerRef:
    name: github-status
  eventSeverity: info
  eventSources:
    - kind: Kustomization
      name: '*'
```

## 🛡️ Multi-Tenancy

### Tenant Isolation

```yaml
# Tenant namespace
apiVersion: v1
kind: Namespace
metadata:
  name: team-a
  labels:
    toolkit.fluxcd.io/tenant: team-a
---
# Service account for tenant
apiVersion: v1
kind: ServiceAccount
metadata:
  name: team-a
  namespace: team-a
---
# Role binding - restrict to namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: team-a-reconciler
  namespace: team-a
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: team-a
    namespace: team-a
---
# Tenant Kustomization
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: team-a-apps
  namespace: team-a
spec:
  serviceAccountName: team-a  # Use tenant SA
  interval: 10m
  sourceRef:
    kind: GitRepository
    name: team-a-repo
    namespace: flux-system
  path: ./apps
  prune: true
  targetNamespace: team-a     # Force namespace
```

## 📊 Monitoring Flux

### Prometheus Metrics

```yaml
# ServiceMonitor for Flux
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: flux-system
  namespace: flux-system
spec:
  endpoints:
    - port: http-prom
  namespaceSelector:
    matchNames:
      - flux-system
  selector:
    matchLabels:
      app.kubernetes.io/part-of: flux
```

### Key Metrics

```yaml
Flux Metrics:
  gotk_reconcile_condition:
    Description: Reconciliation status
    Labels: kind, name, namespace, type, status
    
  gotk_reconcile_duration_seconds:
    Description: Reconciliation duration
    Labels: kind, name, namespace
    
  gotk_suspend_status:
    Description: Suspension status
    Labels: kind, name, namespace
```

## ✅ Best Practices

### Repository Organization
- [ ] Separate infrastructure from applications
- [ ] Use Kustomize overlays for environments
- [ ] Keep secrets in external secret manager
- [ ] Document repository structure

### Security
- [ ] Use RBAC for multi-tenancy
- [ ] Sign commits with GPG
- [ ] Use read-only deploy keys
- [ ] Implement network policies

### Operations
- [ ] Set up notifications for failures
- [ ] Monitor reconciliation metrics
- [ ] Use health checks for dependencies
- [ ] Test changes in staging first

---

**Next Steps**:
- Learn [Argo CD](./argo-cd.md)
- Explore [Helm](./helm.md)
- Master [Kubernetes Operations](./kubernetes-operations.md)


