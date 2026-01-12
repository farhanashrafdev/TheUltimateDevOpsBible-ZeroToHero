# Kubernetes Interview Questions

## 🎯 Introduction

Comprehensive Kubernetes interview questions from basics to CKA/CKAD-level scenarios.

## 📚 Architecture

**Q: What are the components of the Kubernetes control plane?**

**A:**
- **API Server**: RESTful interface, authentication/authorization, etcd access
- **etcd**: Distributed key-value store, cluster state
- **Scheduler**: Assigns pods to nodes based on resources/constraints
- **Controller Manager**: Runs controllers (Node, Replication, Endpoints)
- **Cloud Controller Manager**: Cloud provider integration

**Q: What runs on each worker node?**

**A:**
- **kubelet**: Pod lifecycle management, node registration
- **kube-proxy**: Network rules, service abstraction
- **Container Runtime**: containerd, CRI-O

**Q: How does the Kubernetes scheduler work?**

**A:**
1. Filter nodes (predicates): resources, taints, affinity
2. Score nodes (priorities): resource utilization, spreading
3. Select highest scoring node
4. Bind pod to node

## 📦 Workloads

**Q: What's the difference between Deployments, StatefulSets, and DaemonSets?**

**A:**
- **Deployment**: Stateless apps, declarative updates, rolling restarts
- **StatefulSet**: Stateful apps, stable identity, ordered operations
- **DaemonSet**: One pod per node (monitoring, logging agents)

**Q: Explain Init Containers.**

**A:** Run before main containers:
- Must complete successfully before app containers start
- Run sequentially
- Use cases: Setup, wait for dependencies, fetch secrets

```yaml
initContainers:
  - name: wait-for-db
    image: busybox
    command: ['sh', '-c', 'until nc -z db 5432; do sleep 1; done']
```

**Q: What are Pod disruption budgets?**

**A:** PDBs ensure availability during voluntary disruptions:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: app-pdb
spec:
  minAvailable: 2  # or maxUnavailable: 1
  selector:
    matchLabels:
      app: myapp
```

## 🌐 Networking

**Q: Explain Kubernetes Services and their types.**

**A:**
- **ClusterIP**: Internal-only, default
- **NodePort**: External via node port (30000-32767)
- **LoadBalancer**: Cloud provider LB
- **ExternalName**: DNS CNAME to external service

**Q: How does DNS work in Kubernetes?**

**A:**
- CoreDNS runs as deployment in kube-system
- Pods get DNS config pointing to CoreDNS
- Service discovery: `<service>.<namespace>.svc.cluster.local`
- Pod DNS: `<pod-ip>.<namespace>.pod.cluster.local`

**Q: Explain Ingress and Ingress Controllers.**

**A:**
- **Ingress**: API object defining HTTP routing rules
- **Ingress Controller**: Implements the rules (nginx, traefik)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
spec:
  ingressClassName: nginx
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api
                port:
                  number: 80
```

**Q: Explain Network Policies.**

**A:** Firewall rules for pod communication:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

## 💾 Storage

**Q: Explain PV, PVC, and StorageClass.**

**A:**
- **PersistentVolume (PV)**: Cluster resource, actual storage
- **PersistentVolumeClaim (PVC)**: User request for storage
- **StorageClass**: Dynamic provisioning template

**Q: What are access modes?**

**A:**
- **ReadWriteOnce (RWO)**: Single node read-write
- **ReadOnlyMany (ROX)**: Multiple nodes read-only
- **ReadWriteMany (RWX)**: Multiple nodes read-write

## 🔐 Security

**Q: Explain RBAC components.**

**A:**
- **Role**: Namespace-scoped permissions
- **ClusterRole**: Cluster-wide permissions
- **RoleBinding**: Binds Role to users in namespace
- **ClusterRoleBinding**: Binds ClusterRole cluster-wide

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
```

**Q: What are Security Contexts?**

**A:** Pod/container security settings:

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  readOnlyRootFilesystem: true
  allowPrivilegeEscalation: false
  capabilities:
    drop: ["ALL"]
```

**Q: Explain Pod Security Standards.**

**A:**
- **Privileged**: Unrestricted
- **Baseline**: Prevent known escalations
- **Restricted**: Heavily restricted, best practices

## 🔧 Troubleshooting

**Q: Pod is stuck in Pending. How do you debug?**

**A:**
```bash
kubectl describe pod <pod-name>
# Check Events section for:
# - Insufficient resources
# - Unschedulable (taints/tolerations)
# - PVC not bound
# - Image pull issues
```

**Q: Pod is in CrashLoopBackOff. Debug steps?**

**A:**
```bash
# Check current logs
kubectl logs <pod-name>

# Check previous container logs
kubectl logs <pod-name> --previous

# Check pod events
kubectl describe pod <pod-name>

# Verify command/args
kubectl get pod <pod-name> -o yaml
```

**Q: How do you debug networking issues between pods?**

**A:**
```bash
# Check pod IPs
kubectl get pods -o wide

# Test connectivity from pod
kubectl exec -it <pod> -- ping <other-pod-ip>
kubectl exec -it <pod> -- curl <service-name>

# Check DNS
kubectl exec -it <pod> -- nslookup <service>

# Check network policies
kubectl get networkpolicies

# Check service endpoints
kubectl get endpoints <service>
```

## 🎯 Scenario Questions

**Q: Design a highly available web application deployment.**

**A:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchLabels:
                  app: web
              topologyKey: kubernetes.io/hostname
      containers:
        - name: web
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "256Mi"
              cpu: "200m"
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 30
          readinessProbe:
            httpGet:
              path: /ready
              port: 8080
```

**Q: How would you implement blue-green deployment?**

**A:**
1. Deploy new version with different label
2. Test new version internally
3. Switch service selector to new version
4. Keep old version for quick rollback

```bash
# Deploy green
kubectl apply -f deployment-green.yaml

# Test green
kubectl port-forward svc/web-green 8080

# Switch traffic
kubectl patch svc web -p '{"spec":{"selector":{"version":"green"}}}'
```

---

**Next**: Review [AWS Interview](./aws.md) questions.

