# Argo CD

## 🎯 Introduction

Argo CD is a declarative GitOps continuous delivery tool for Kubernetes, automatically syncing applications from Git repositories.

## 📚 Core Concepts

### GitOps with Argo CD
- Git as source of truth
- Automatic synchronization
- Multi-cluster support
- Rollback via Git

## 📝 Application Definition

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
spec:
  project: default
  source:
    repoURL: https://github.com/user/repo.git
    targetRevision: main
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

## 🔧 Key Features

### Sync Policies
- **Manual**: User-triggered sync
- **Automated**: Automatic sync
- **Self-heal**: Auto-correct drift

### Multi-cluster
```yaml
destination:
  server: https://cluster2.example.com
  namespace: production
```

## ✅ Best Practices

- Use Git as source of truth
- Enable automated sync
- Monitor sync status
- Use application sets
- Secure access
- Implement RBAC

---

**Next**: Learn Argo Workflows for workflow orchestration.

