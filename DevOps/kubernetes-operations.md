# Kubernetes Operations: Complete Operational Guide

## 🎯 Introduction

Kubernetes operations involve managing, scaling, monitoring, and maintaining Kubernetes clusters and applications in production. This guide covers everything you need to operate Kubernetes effectively.

### Operational Challenges

- **Scaling**: Handle traffic spikes
- **High Availability**: Ensure uptime
- **Resource Management**: Optimize resource usage
- **Updates**: Zero-downtime deployments
- **Monitoring**: Track application health
- **Troubleshooting**: Debug issues quickly
- **Backup & Recovery**: Protect data

## 📊 Resource Management

### Requests and Limits

**Understanding Resources**:

```yaml
resources:
  requests:
    memory: "128Mi"    # Guaranteed
    cpu: "250m"        # Guaranteed (0.25 cores)
  limits:
    memory: "256Mi"     # Maximum
    cpu: "500m"        # Maximum (0.5 cores)
```

**CPU Units**:
- `1` = 1 CPU core
- `1000m` = 1 CPU core
- `500m` = 0.5 CPU core
- `250m` = 0.25 CPU core

**Memory Units**:
- `128Mi` = 128 Mebibytes
- `1Gi` = 1 Gibibyte
- `512M` = 512 Megabytes

### Complete Resource Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: app
        image: myapp:latest
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        # Resource quotas help scheduler
```

### Resource Quotas

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: production
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    persistentvolumeclaims: "10"
    pods: "10"
```

### Limit Ranges

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
  namespace: production
spec:
  limits:
  - default:
      memory: "512Mi"
      cpu: "500m"
    defaultRequest:
      memory: "256Mi"
      cpu: "250m"
    type: Container
  - max:
      memory: "1Gi"
      cpu: "1"
    min:
      memory: "128Mi"
      cpu: "100m"
    type: Container
```

## 🔄 Scaling

### Manual Scaling

```bash
# Scale deployment
kubectl scale deployment web-app --replicas=5

# Scale statefulset
kubectl scale statefulset database --replicas=3

# Scale with file
kubectl scale --replicas=5 -f deployment.yaml
```

### Horizontal Pod Autoscaler (HPA)

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
      - type: Pods
        value: 4
        periodSeconds: 15
      selectPolicy: Max
```

**HPA Commands**:
```bash
# Create HPA
kubectl autoscale deployment web-app --min=2 --max=10 --cpu-percent=70

# Get HPA
kubectl get hpa

# Describe HPA
kubectl describe hpa web-app-hpa
```

### Vertical Pod Autoscaler (VPA)

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: web-app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
    - containerName: app
      minAllowed:
        cpu: 100m
        memory: 128Mi
      maxAllowed:
        cpu: 1
        memory: 1Gi
```

### Cluster Autoscaler

```yaml
# Node autoscaling (cloud-specific)
# Automatically adds/removes nodes based on demand
```

## 🔄 Rolling Updates

### Deployment Strategies

#### Rolling Update (Default)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1          # Can have 1 extra pod
      maxUnavailable: 0    # Always have pods available
  template:
    spec:
      containers:
      - name: app
        image: myapp:v2.0.0
```

**Rolling Update Process**:
1. Create new pod with new version
2. Wait for new pod to be ready
3. Terminate old pod
4. Repeat until all pods updated

#### Recreate Strategy

```yaml
spec:
  strategy:
    type: Recreate
```

**Process**:
1. Terminate all old pods
2. Create all new pods

### Rollout Management

```bash
# Check rollout status
kubectl rollout status deployment/web-app

# View rollout history
kubectl rollout history deployment/web-app

# View specific revision
kubectl rollout history deployment/web-app --revision=2

# Rollback to previous version
kubectl rollout undo deployment/web-app

# Rollback to specific revision
kubectl rollout undo deployment/web-app --to-revision=2

# Pause rollout
kubectl rollout pause deployment/web-app

# Resume rollout
kubectl rollout resume deployment/web-app

# Restart deployment
kubectl rollout restart deployment/web-app
```

### Canary Deployment

```yaml
# Deploy canary version
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app-canary
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: web-app
        version: canary
    spec:
      containers:
      - name: app
        image: myapp:v2.0.0-canary
---
# Route traffic to canary
apiVersion: v1
kind: Service
metadata:
  name: web-app
spec:
  selector:
    app: web-app
    version: canary  # Initially route to canary
  ports:
  - port: 80
```

### Blue-Green Deployment

```yaml
# Blue deployment (current)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app-blue
spec:
  replicas: 5
  template:
    metadata:
      labels:
        app: web-app
        version: blue
---
# Green deployment (new)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app-green
spec:
  replicas: 5
  template:
    metadata:
      labels:
        app: web-app
        version: green
---
# Service switches between blue/green
apiVersion: v1
kind: Service
metadata:
  name: web-app
spec:
  selector:
    app: web-app
    version: green  # Switch to green
```

## 🏥 Health Checks

### Liveness Probe

**Detects if container is running**

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
    httpHeaders:
    - name: Custom-Header
      value: Awesome
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  successThreshold: 1
  failureThreshold: 3
```

**Probe Types**:
- **httpGet**: HTTP request
- **tcpSocket**: TCP connection
- **exec**: Command execution

### Readiness Probe

**Detects if container is ready to serve traffic**

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
  timeoutSeconds: 3
  successThreshold: 1
  failureThreshold: 3
```

### Startup Probe

**For slow-starting containers**

```yaml
startupProbe:
  httpGet:
    path: /startup
    port: 8080
  failureThreshold: 30
  periodSeconds: 10
```

**Complete Health Check Example**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  template:
    spec:
      containers:
      - name: app
        image: myapp:latest
        ports:
        - containerPort: 8080
        
        # Startup probe (for slow starters)
        startupProbe:
          httpGet:
            path: /startup
            port: 8080
          failureThreshold: 30
          periodSeconds: 10
        
        # Readiness probe (traffic routing)
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
        
        # Liveness probe (container restart)
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
```

## 📦 ConfigMaps and Secrets

### ConfigMaps

**Non-confidential configuration data**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: production
data:
  # Simple key-value
  database_url: "postgresql://db:5432/mydb"
  log_level: "info"
  
  # File content
  config.yaml: |
    server:
      port: 8080
      host: 0.0.0.0
    database:
      host: db
      port: 5432
  
  # JSON
  app.json: |
    {
      "name": "myapp",
      "version": "1.0.0"
    }
```

**Using ConfigMap**:

```yaml
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: app
    image: myapp:latest
    # Environment variables
    envFrom:
    - configMapRef:
        name: app-config
    # Or specific keys
    env:
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database_url
    # Volume mount
    volumeMounts:
    - name: config
      mountPath: /etc/config
  volumes:
  - name: config
    configMap:
      name: app-config
```

### Secrets

**Sensitive data**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
  namespace: production
type: Opaque
data:
  username: YWRtaW4=        # base64 encoded
  password: c2VjcmV0MTIz     # base64 encoded
stringData:
  api_key: "secret-key"      # Automatically encoded
```

**Using Secrets**:

```yaml
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: app
    image: myapp:latest
    env:
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: username
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: password
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: app-secret
```

## 🔄 Jobs and CronJobs

### Jobs

**One-time tasks**

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: backup-job
spec:
  completions: 1
  parallelism: 1
  backoffLimit: 3
  activeDeadlineSeconds: 300
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: backup
        image: backup-tool:latest
        command: ["/bin/sh"]
        args: ["-c", "backup.sh"]
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
```

### CronJobs

**Scheduled tasks**

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: daily-backup
spec:
  schedule: "0 2 * * *"  # Daily at 2 AM
  timeZone: "America/New_York"
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  concurrencyPolicy: Forbid
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: backup
            image: backup-tool:latest
            command: ["backup.sh"]
```

**Cron Schedule Format**:
```
┌───────────── minute (0 - 59)
│ ┌───────────── hour (0 - 23)
│ │ ┌───────────── day of month (1 - 31)
│ │ │ ┌───────────── month (1 - 12)
│ │ │ │ ┌───────────── day of week (0 - 6)
│ │ │ │ │
* * * * *
```

## 🔍 Monitoring and Observability

### Resource Usage

```bash
# Install metrics server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# View resource usage
kubectl top nodes
kubectl top pods
kubectl top pods --containers
kubectl top pods --sort-by=memory
kubectl top pods --sort-by=cpu
```

### Events

```bash
# View events
kubectl get events
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl get events --field-selector involvedObject.kind=Pod
kubectl get events -n production
```

### Logs

```bash
# Pod logs
kubectl logs pod-name
kubectl logs -f pod-name
kubectl logs --previous pod-name
kubectl logs -l app=nginx

# Deployment logs
kubectl logs deployment/web-app
kubectl logs -f deployment/web-app

# Multi-container pod
kubectl logs pod-name -c container-name
```

## 🛠️ Maintenance Operations

### Node Maintenance

```bash
# Drain node
kubectl drain node-name --ignore-daemonsets --delete-emptydir-data

# Cordon node (prevent scheduling)
kubectl cordon node-name

# Uncordon node
kubectl uncordon node-name
```

### Pod Disruption Budget

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: web-app-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: web-app
```

### Cluster Maintenance

```bash
# Backup etcd
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.crt \
  --cert=/etc/etcd/etcd.crt \
  --key=/etc/etcd/etcd.key

# Restore etcd
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-snapshot.db
```

## ✅ Best Practices

### 1. Set Resource Requests and Limits

```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "250m"
  limits:
    memory: "512Mi"
    cpu: "500m"
```

### 2. Use Health Checks

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
```

### 3. Implement Proper Logging

```yaml
# Use structured logging
# Log to stdout/stderr
# Use log aggregation
```

### 4. Monitor Everything

```yaml
# Use Prometheus, Grafana
# Monitor resource usage
# Set up alerts
```

### 5. Use Namespaces

```yaml
# Organize by environment
# Organize by team
# Use resource quotas
```

### 6. Implement RBAC

```yaml
# Least privilege
# Service accounts
# Role-based access
```

### 7. Regular Backups

```bash
# Backup etcd
# Backup persistent volumes
# Backup configurations
```

## ✅ Mastery Checklist

- [ ] Manage resource requests and limits
- [ ] Implement HPA for autoscaling
- [ ] Perform rolling updates
- [ ] Use health checks effectively
- [ ] Manage ConfigMaps and Secrets
- [ ] Create Jobs and CronJobs
- [ ] Monitor resource usage
- [ ] Perform node maintenance
- [ ] Implement Pod Disruption Budgets
- [ ] Backup and restore clusters

---

**Next Steps**:
- Learn [Kubernetes Networking](./kubernetes-networking.md)
- Explore [Kubernetes Storage](./kubernetes-storage.md)
- Master [Monitoring & Observability](./monitoring-observability.md)

**Remember**: Kubernetes operations require careful resource management, proper health checks, and continuous monitoring. Start with basics, implement best practices, and always have a backup strategy.
