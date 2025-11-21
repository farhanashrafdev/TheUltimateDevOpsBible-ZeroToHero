# Infrastructure Orchestrators: Crossplane & Humanitec

## 🎯 Introduction

How do developers provision infrastructure?
- **Old Way**: Open a Jira ticket -> Ops runs Terraform -> 3 days later -> DB ready.
- **Platform Way**: Dev defines intent -> Orchestrator provisions -> 5 mins later -> DB ready.

## 🛠️ Tools

### 1. Crossplane (Kubernetes Native)

**Concept**: Build your own Cloud API on Kubernetes.
- **Composite Resource (XRD)**: Define your abstract API (e.g., `PostgresDB`).
- **Composition**: Map abstract API to real cloud resources (e.g., `AWS RDS`).

**Dev Experience**:
```yaml
apiVersion: database.myorg.com/v1alpha1
kind: PostgresDB
metadata:
  name: my-db
  namespace: my-app
spec:
  storageGB: 20
```

**Platform Engineer's Job**:
- Write the XRD and Composition that maps `PostgresDB` -> `AWS RDS` + `Security Group` + `Secret`.

### 2. Humanitec (Platform Orchestrator)

**Concept**: Dynamic Configuration Management.
- **Score**: Workload specification (platform agnostic).
- **Driver**: Provisioning logic.

**Dev Experience (score.yaml)**:
```yaml
apiVersion: score.dev/v1b1
metadata:
  name: my-workload
containers:
  backend:
    image: my-image
resources:
  db:
    type: postgres
```

---

## 🆚 Comparison

| Feature | Crossplane | Humanitec | Terraform |
|:---|:---|:---|:---|
| **Type** | Control Plane | IDP Engine | IaC Tool |
| **State** | Continuous Reconciliation | Deployment based | State file |
| **Interface** | Kubernetes CRDs | API / UI / CLI | HCL |
| **Drift Detection** | Auto-correction | Deployment based | Plan/Apply |

---

## 🎯 Why Orchestrators?

1. **Abstraction**: Devs don't need to know AWS/Azure details.
2. **Standardization**: Enforce tagging, security, sizing.
3. **Self-Service**: No tickets.
4. **Portability**: Swap AWS for Azure without changing Dev manifests.

---

## ✅ Best Practices

- [ ] Hide complexity, expose only what Devs need (T-shirt sizing)
- [ ] Use Crossplane if you are 100% Kubernetes
- [ ] Use Terraform for the "base" (VPC, Clusters)
- [ ] Use Orchestrators for "app dependencies" (DBs, Queues)

---

**Next**: [Developer Experience](./developer-experience.md).
