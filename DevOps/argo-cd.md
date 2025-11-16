# Argo CD: Complete GitOps Continuous Delivery Guide

## 🎯 Introduction

Argo CD is a declarative, GitOps continuous delivery tool for Kubernetes. It automatically syncs and manages applications from Git repositories to Kubernetes clusters, ensuring your cluster state matches your Git repository.

### Why Argo CD?

**Key Benefits**:
- **GitOps**: Git as single source of truth
- **Automatic Sync**: Continuous synchronization
- **Multi-Cluster**: Manage multiple clusters
- **Rollback**: Easy rollback via Git
- **RBAC**: Fine-grained access control
- **Web UI**: Visual application management
- **Health Monitoring**: Application health status
- **Sync Policies**: Flexible sync strategies

### GitOps Principles

1. **Declarative**: Everything described as code
2. **Version Controlled**: All config in Git
3. **Automated**: Automated sync and deployment
4. **Observable**: Monitor and alert on drift

## 📚 Core Concepts

### Argo CD Architecture

```
Git Repository (Source of Truth)
    │
    ▼
Argo CD Application Controller
    │
    ├───► Kubernetes Cluster 1
    ├───► Kubernetes Cluster 2
    └───► Kubernetes Cluster 3
```

### Key Components

- **Application Controller**: Monitors and syncs applications
- **API Server**: REST API and web UI
- **Repo Server**: Fetches and caches Git repositories
- **Application CRD**: Kubernetes custom resource

## 📝 Complete Examples

### Basic Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  
  source:
    repoURL: https://github.com/user/repo.git
    targetRevision: main
    path: k8s/overlays/production
    
  destination:
    server: https://kubernetes.default.svc
    namespace: production
    
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
    syncOptions:
      - CreateNamespace=true
      - PrunePropagationPolicy=foreground
      - PruneLast=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
```

### Multi-Environment Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app-staging
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/user/repo.git
    targetRevision: develop
    path: k8s/overlays/staging
  destination:
    server: https://kubernetes.default.svc
    namespace: staging
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
---
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app-production
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/user/repo.git
    targetRevision: main
    path: k8s/overlays/production
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: false  # Manual sync for production
      selfHeal: true
```

### Helm Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-helm-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://charts.example.com
    chart: myapp
    targetRevision: 1.0.0
    helm:
      valueFiles:
        - values-production.yaml
      values: |
        replicaCount: 3
        image:
          tag: v1.0.0
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### Kustomize Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-kustomize-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/user/repo.git
    targetRevision: main
    path: k8s/base
    kustomize:
      images:
        - myapp:v1.0.0
      commonLabels:
        app: myapp
        env: production
  destination:
    server: https://kubernetes.default.svc
    namespace: production
```

### Multi-Cluster Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app-cluster1
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/user/repo.git
    targetRevision: main
    path: k8s
  destination:
    server: https://cluster1.example.com
    namespace: production
---
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app-cluster2
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/user/repo.git
    targetRevision: main
    path: k8s
  destination:
    server: https://cluster2.example.com
    namespace: production
```

## 🔧 Advanced Features

### Application Sets

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: my-apps
  namespace: argocd
spec:
  generators:
    - list:
        elements:
          - cluster: production
            url: https://prod-cluster.example.com
          - cluster: staging
            url: https://staging-cluster.example.com
  template:
    metadata:
      name: '{{cluster}}-app'
    spec:
      project: default
      source:
        repoURL: https://github.com/user/repo.git
        targetRevision: main
        path: k8s
      destination:
        server: '{{url}}'
        namespace: production
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

### Sync Policies

```yaml
syncPolicy:
  # Automated sync
  automated:
    prune: true          # Delete resources not in Git
    selfHeal: true       # Auto-correct drift
    allowEmpty: false    # Don't sync if no resources
  
  # Sync options
  syncOptions:
    - CreateNamespace=true
    - PrunePropagationPolicy=foreground
    - PruneLast=true
    - ApplyOutOfSyncOnly=true
  
  # Retry configuration
  retry:
    limit: 5
    backoff:
      duration: 5s
      factor: 2
      maxDuration: 3m
```

### Health Checks

```yaml
spec:
  source:
    repoURL: https://github.com/user/repo.git
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  # Custom health checks
  # Argo CD automatically detects health based on resource status
```

### Resource Hooks

```yaml
# Pre-sync hook
apiVersion: batch/v1
kind: Job
metadata:
  name: pre-sync-job
  annotations:
    argocd.argoproj.io/hook: PreSync
    argocd.argoproj.io/hook-delete-policy: BeforeHookCreation
spec:
  template:
    spec:
      containers:
        - name: pre-sync
          image: busybox
          command: [sh, -c, 'echo Pre-sync hook']
      restartPolicy: Never

# Post-sync hook
apiVersion: batch/v1
kind: Job
metadata:
  name: post-sync-job
  annotations:
    argocd.argoproj.io/hook: PostSync
    argocd.argoproj.io/hook-delete-policy: HookSucceeded
spec:
  template:
    spec:
      containers:
        - name: post-sync
          image: busybox
          command: [sh, -c, 'echo Post-sync hook']
      restartPolicy: Never
```

## 🛠️ CLI Commands

### Application Management

```bash
# List applications
argocd app list
argocd app get my-app

# Create application
argocd app create my-app \
  --repo https://github.com/user/repo.git \
  --path k8s \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace production

# Sync application
argocd app sync my-app
argocd app sync my-app --prune
argocd app sync my-app --dry-run

# Get application status
argocd app get my-app
argocd app history my-app
argocd app diff my-app

# Rollback
argocd app rollback my-app
argocd app rollback my-app <revision>

# Delete application
argocd app delete my-app
```

### Sync Operations

```bash
# Manual sync
argocd app sync my-app

# Sync with prune
argocd app sync my-app --prune

# Sync specific resources
argocd app sync my-app --resource Deployment:my-deployment

# Hard refresh
argocd app get my-app --hard-refresh
```

### Repository Management

```bash
# Add repository
argocd repo add https://github.com/user/repo.git \
  --type git \
  --name my-repo

# List repositories
argocd repo list

# Remove repository
argocd repo rm https://github.com/user/repo.git
```

## 🎯 Best Practices

### 1. Use Projects for Organization

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: production
  namespace: argocd
spec:
  description: Production applications
  sourceRepos:
    - 'https://github.com/user/repo.git'
  destinations:
    - namespace: production
      server: https://kubernetes.default.svc
  clusterResourceWhitelist:
    - group: '*'
      kind: '*'
```

### 2. Enable Automated Sync Carefully

```yaml
# Staging - can be automated
syncPolicy:
  automated:
    prune: true
    selfHeal: true

# Production - manual sync
syncPolicy:
  automated:
    prune: false
    selfHeal: true
```

### 3. Use Sync Windows

```yaml
syncPolicy:
  syncWindows:
    - kind: allow
      schedule: '10 1 * * *'  # Allow sync at 1:10 AM
      duration: 1h
      applications:
        - '*'
    - kind: deny
      schedule: '* * * * *'   # Deny all other times
      applications:
        - production-*
```

### 4. Implement RBAC

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-rbac-cm
  namespace: argocd
data:
  policy.csv: |
    p, role:admin, applications, *, */*, allow
    p, role:developer, applications, get, */*, allow
    p, role:developer, applications, sync, staging/*, allow
```

## 🔍 Monitoring

### Application Health

```bash
# Check health status
argocd app get my-app

# Health states:
# - Healthy: All resources healthy
# - Progressing: Resources being created/updated
# - Degraded: Some resources unhealthy
# - Suspended: Application suspended
# - Missing: Resources missing
# - Unknown: Health status unknown
```

### Sync Status

```bash
# Sync states:
# - Synced: Cluster matches Git
# - OutOfSync: Cluster differs from Git
# - Unknown: Status unknown
```

## ✅ Mastery Checklist

- [ ] Install and configure Argo CD
- [ ] Create applications
- [ ] Configure sync policies
- [ ] Use Application Sets
- [ ] Implement RBAC
- [ ] Set up multi-cluster
- [ ] Use hooks for custom logic
- [ ] Monitor application health
- [ ] Rollback applications
- [ ] Integrate with CI/CD

---

**Next Steps**:
- Learn [Argo Workflows](./argo-workflows.md) for workflow orchestration
- Explore [Helm](./helm.md) for package management
- Master [Kubernetes Operations](./kubernetes-operations.md)

**Remember**: Argo CD is GitOps in action. Keep Git as your source of truth, use automated sync carefully, and always monitor application health. Start with simple applications and gradually add complexity.
