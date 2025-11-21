# Kubernetes Interview Questions: Complete Guide

## 🎯 Introduction

This comprehensive guide covers Kubernetes interview questions from basic concepts to advanced scenarios, helping you ace your Kubernetes interviews.

## 📚 Core Concepts

### Explain Kubernetes Architecture

**Answer**: Kubernetes has two main components:

**Control Plane (Master)**:
- **API Server**: Central management point, REST API
- **etcd**: Distributed key-value store (cluster state)
- **Scheduler**: Assigns pods to nodes
- **Controller Manager**: Runs controllers (Deployment, ReplicaSet, etc.)

**Worker Nodes**:
- **kubelet**: Agent that communicates with API server
- **kube-proxy**: Network proxy, maintains network rules
- **Container Runtime**: Docker, containerd, CRI-O

**Communication**:
- Control plane → Nodes: API server → kubelet
- Nodes → Control plane: kubelet → API server
- Pods → Pods: Via kube-proxy and CNI

### What is a Pod?

**Answer**: Pod is the smallest deployable unit in Kubernetes:

**Characteristics**:
- Contains one or more containers
- Containers share network namespace (same IP)
- Containers share storage volumes
- Containers can communicate via localhost
- Ephemeral (can be recreated)

**Why Pods?**:
- Group tightly coupled containers
- Share resources efficiently
- Atomic unit of deployment

**Example**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.21
  - name: sidecar
    image: busybox
```

### What is a Deployment?

**Answer**: Deployment manages pod replicas and provides:

**Features**:
- Declarative updates
- Rolling updates
- Rollback capabilities
- Scaling
- Self-healing

**How it works**:
1. Creates ReplicaSet
2. ReplicaSet creates Pods
3. Monitors desired vs actual state
4. Maintains desired number of replicas

**Example**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
```

### Explain Kubernetes Services

**Answer**: Services provide stable network access to pods:

**Problem**: Pods are ephemeral, IPs change
**Solution**: Services provide stable endpoints

**Service Types**:
1. **ClusterIP**: Internal only (default)
2. **NodePort**: External via node IP:port
3. **LoadBalancer**: External via cloud LB
4. **ExternalName**: Maps to external DNS

**How Services Work**:
- Selector matches pod labels
- Endpoints controller creates Endpoints object
- kube-proxy implements service (iptables/ipvs)

**Example**:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP
```

### What are ConfigMaps and Secrets?

**Answer**:

**ConfigMaps**:
- Store non-confidential configuration
- Key-value pairs or files
- Can be mounted as volumes or env vars

**Secrets**:
- Store sensitive data (base64 encoded)
- Passwords, tokens, keys
- Encrypted at rest (with encryption config)

**Usage**:
```yaml
env:
- name: DB_URL
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: database_url
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: app-secret
      key: password
```

**Best Practices**:
- Never commit secrets
- Use external secret managers
- Rotate secrets regularly
- Use RBAC to restrict access

### What is Ingress?

**Answer**: Ingress provides HTTP/HTTPS routing to services:

**Features**:
- URL-based routing
- SSL/TLS termination
- Load balancing
- Name-based virtual hosting

**Requires**: Ingress Controller (NGINX, Traefik, etc.)

**Example**:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
spec:
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-service
            port:
              number: 80
```

## 🔧 Advanced Topics

### Explain Network Policies

**Answer**: Network Policies control pod-to-pod communication:

**Purpose**:
- Micro-segmentation
- Security isolation
- Traffic filtering

**How it works**:
- PodSelector: Selects pods
- PolicyTypes: Ingress, Egress, or both
- Rules: Define allowed traffic

**Example**:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: app-policy
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
```

**Note**: Requires CNI plugin that supports Network Policies (Calico, Cilium, etc.)

### Explain PersistentVolumes

**Answer**: PersistentVolumes provide cluster-wide storage:

**Components**:
- **PV (PersistentVolume)**: Cluster resource (like a node)
- **PVC (PersistentVolumeClaim)**: Request for storage (like a pod)
- **StorageClass**: Defines provisioner and parameters

**Lifecycle**:
1. Admin creates PV (or dynamic via StorageClass)
2. User creates PVC
3. PVC binds to PV
4. Pod uses PVC

**Example**:
```yaml
# StorageClass
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3

# PVC
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: fast-ssd
```

### What is a StatefulSet?

**Answer**: StatefulSet manages stateful applications:

**Features**:
- Stable network identity (hostname)
- Stable storage (persistent volumes)
- Ordered deployment
- Ordered scaling
- Ordered updates

**Use Cases**:
- Databases
- Message queues
- Stateful applications

**Example**:
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

### Explain Horizontal Pod Autoscaler (HPA)

**Answer**: HPA automatically scales pods based on metrics:

**How it works**:
1. Monitors metrics (CPU, memory, custom)
2. Compares to target
3. Scales up/down replicas
4. Adjusts gradually

**Example**:
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

**Requirements**: Metrics Server or custom metrics adapter

### What is a DaemonSet?

**Answer**: DaemonSet ensures all (or specific) nodes run a copy of a pod:

**Use Cases**:
- Log collection (Fluentd)
- Node monitoring (Prometheus node exporter)
- Network plugins (kube-proxy)
- Storage daemons

**Example**:
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluentd:latest
```

### Explain Jobs and CronJobs

**Answer**:

**Job**: Runs pods until completion (one-time task)

**CronJob**: Runs Jobs on a schedule

**Example**:
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup-job
spec:
  schedule: "0 2 * * *"  # Daily at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: backup-tool:latest
          restartPolicy: OnFailure
```

## 🔒 Security

### Explain RBAC

**Answer**: Role-Based Access Control manages permissions:

**Components**:
- **Role**: Permissions within namespace
- **ClusterRole**: Permissions cluster-wide
- **RoleBinding**: Binds role to users/groups/service accounts
- **ClusterRoleBinding**: Cluster-wide binding

**Example**:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
subjects:
- kind: User
  name: developer
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### Explain Pod Security Standards

**Answer**: Pod Security Standards define security policies:

**Levels**:
- **Privileged**: No restrictions
- **Baseline**: Minimal restrictions
- **Restricted**: Strong restrictions

**Example**:
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

## 📝 Scenario Questions

### Pod is not starting. How do you troubleshoot?

**Answer**:
1. **Check pod status**: `kubectl get pods`
2. **Describe pod**: `kubectl describe pod pod-name`
3. **Check events**: `kubectl get events --sort-by=.metadata.creationTimestamp`
4. **View logs**: `kubectl logs pod-name` (if container started)
5. **Check previous logs**: `kubectl logs pod-name --previous`
6. **Verify image**: Check if image exists and is accessible
7. **Check resources**: CPU/memory limits
8. **Verify configuration**: ConfigMaps, Secrets
9. **Check node**: `kubectl describe node node-name`
10. **Check network**: Network policies, service connectivity

**Common Issues**:
- Image pull errors
- Resource constraints
- Configuration errors
- Network policies blocking
- Node issues

### Design a scalable Kubernetes architecture

**Answer**:
1. **Cluster Design**:
   - Multi-AZ deployment
   - Control plane HA
   - Worker node groups

2. **Application Design**:
   - Deployments with HPA
   - Services for load balancing
   - Ingress for external access

3. **Storage**:
   - StorageClasses for dynamic provisioning
   - PersistentVolumes for stateful apps

4. **Networking**:
   - CNI plugin (Calico, Cilium)
   - Network policies
   - Service mesh (optional)

5. **Monitoring**:
   - Prometheus + Grafana
   - Log aggregation
   - Distributed tracing

6. **Security**:
   - RBAC
   - Pod security standards
   - Network policies
   - Image scanning

7. **CI/CD**:
   - GitOps (Argo CD)
   - Automated deployments

### How do you perform a zero-downtime deployment?

**Answer**:
1. **Rolling Update** (default):
   ```yaml
   strategy:
     type: RollingUpdate
     rollingUpdate:
       maxSurge: 1
       maxUnavailable: 0
   ```

2. **Blue-Green**:
   - Deploy new version alongside old
   - Switch traffic instantly
   - Keep old version for rollback

3. **Canary**:
   - Deploy to small percentage
   - Monitor metrics
   - Gradually increase
   - Rollback if issues

4. **Best Practices**:
   - Health checks (readiness, liveness)
   - Proper resource limits
   - Monitoring during rollout
   - Rollback plan

### How do you handle secrets in Kubernetes?

**Answer**:
1. **Native Secrets** (basic):
   - Base64 encoded
   - Encrypted at rest (with encryption config)
   - RBAC for access control

2. **External Secret Managers**:
   - HashiCorp Vault
   - AWS Secrets Manager
   - Azure Key Vault
   - External Secrets Operator

3. **Best Practices**:
   - Never commit secrets
   - Use external secret managers
   - Rotate regularly
   - Least privilege access
   - Audit secret access

## ✅ Key Areas to Master

- [ ] Pod lifecycle
- [ ] Deployments and ReplicaSets
- [ ] Services and networking
- [ ] ConfigMaps and Secrets
- [ ] Storage (PVs, PVCs, StorageClasses)
- [ ] Ingress and networking
- [ ] Security (RBAC, Network Policies)
- [ ] Autoscaling (HPA, VPA, Cluster Autoscaler)
- [ ] StatefulSets and Jobs
- [ ] Troubleshooting

---

**Remember**: Kubernetes interviews test deep understanding of concepts and practical troubleshooting skills. Be prepared to explain how things work, not just what they are. Practice with real clusters and scenarios.
