# Kubernetes Operations

## 🎯 Operational Concepts

### Resource Management
- **Requests**: Minimum resources
- **Limits**: Maximum resources
- **Quotas**: Namespace limits
- **Priority**: Pod scheduling priority

### Scaling
```yaml
apiVersion: apps/v1
kind: Deployment
spec:
  replicas: 3
```

```bash
# Manual scaling
kubectl scale deployment nginx --replicas=5

# Horizontal Pod Autoscaler
kubectl autoscale deployment nginx --min=2 --max=10 --cpu-percent=80
```

### Rolling Updates
```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
```

### Health Checks
```yaml
spec:
  containers:
  - name: nginx
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 30
      periodSeconds: 10
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
```

## 🔧 Advanced Operations

### ConfigMaps
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  config.yaml: |
    key: value
```

### Secrets
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  password: base64encoded
```

### Jobs and CronJobs
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: backup-job
spec:
  template:
    spec:
      containers:
      - name: backup
        image: backup:latest
      restartPolicy: Never
```

## ✅ Best Practices

- Set resource requests and limits
- Use health checks
- Implement proper logging
- Monitor resource usage
- Use namespaces
- Implement RBAC
- Regular backups

---

**Next**: Learn Kubernetes networking.

