# Production Practices: Complete Guide to Production Excellence

## 🎯 Introduction

Running applications in production requires careful planning, robust practices, and continuous improvement. This guide covers everything you need to operate production systems successfully.

### Production Readiness Checklist

- [ ] High availability configured
- [ ] Monitoring and alerting set up
- [ ] Backup and recovery tested
- [ ] Security hardened
- [ ] Documentation complete
- [ ] Runbooks created
- [ ] Disaster recovery plan
- [ ] Capacity planning done
- [ ] Performance tested
- [ ] Incident response ready

## 🚀 Deployment Strategies

### Blue-Green Deployment

**Instant switch between environments**

```yaml
# Blue (current)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-blue
spec:
  replicas: 5
  template:
    metadata:
      labels:
        app: myapp
        version: blue
    spec:
      containers:
      - name: app
        image: myapp:v1.0.0

---
# Green (new)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-green
spec:
  replicas: 5
  template:
    metadata:
      labels:
        app: myapp
        version: green
    spec:
      containers:
      - name: app
        image: myapp:v2.0.0

---
# Service switches between blue/green
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector:
    app: myapp
    version: green  # Switch to green
  ports:
  - port: 80
```

**Benefits**:
- Zero downtime
- Instant rollback
- Easy testing

**Drawbacks**:
- Requires double resources
- Database migration complexity

### Canary Deployment

**Gradual traffic shift**

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: canary-deployment
spec:
  hosts:
  - myapp
  http:
  - route:
    - destination:
        host: myapp
        subset: stable
      weight: 90
    - destination:
        host: myapp
        subset: canary
      weight: 10
```

**Process**:
1. Deploy canary (10% traffic)
2. Monitor metrics
3. Gradually increase (25%, 50%, 100%)
4. Rollback if issues

### Rolling Update

**Incremental pod replacement**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    spec:
      containers:
      - name: app
        image: myapp:v2.0.0
```

### Feature Flags

**Toggle features without deployment**

```python
from featureflags import FeatureFlags

flags = FeatureFlags(api_key="your-key")

# In code
if flags.is_enabled("new-feature", user_id):
    # New feature code
    new_feature()
else:
    # Old feature code
    old_feature()
```

## 🏗️ High Availability

### Multi-Region Deployment

```yaml
# Region 1
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-us-east
spec:
  replicas: 3
  template:
    spec:
      nodeSelector:
        region: us-east-1

---
# Region 2
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-us-west
spec:
  replicas: 3
  template:
    spec:
      nodeSelector:
        region: us-west-2
```

### Health Checks

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    spec:
      containers:
      - name: app
        image: myapp:latest
        # Startup probe
        startupProbe:
          httpGet:
            path: /startup
            port: 8080
          failureThreshold: 30
          periodSeconds: 10
        
        # Readiness probe
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
        
        # Liveness probe
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
          failureThreshold: 3
```

### Pod Disruption Budget

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: myapp-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: myapp
```

### Auto-Scaling

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 3
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
```

## 💾 Backup & Recovery

### Database Backups

```bash
# PostgreSQL backup
pg_dump -h db-host -U user -d database > backup.sql

# MySQL backup
mysqldump -h db-host -u user -p database > backup.sql

# Automated backup script
#!/bin/bash
BACKUP_DIR="/backups"
DATE=$(date +%Y%m%d_%H%M%S)
pg_dump -h db-host -U user -d database | gzip > "$BACKUP_DIR/backup_$DATE.sql.gz"
# Keep last 7 days
find "$BACKUP_DIR" -name "backup_*.sql.gz" -mtime +7 -delete
```

### Kubernetes Backup

```bash
# Backup etcd
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.crt \
  --cert=/etc/etcd/etcd.crt \
  --key=/etc/etcd/etcd.key

# Backup persistent volumes
kubectl exec -n production postgres-pod -- pg_dump database > backup.sql
```

### Disaster Recovery

**RTO (Recovery Time Objective)**: Target time to restore
**RPO (Recovery Point Objective)**: Maximum acceptable data loss

```yaml
# Disaster Recovery Plan
1. Identify critical systems
2. Define RTO and RPO
3. Implement backups
4. Test restore procedures
5. Document recovery steps
6. Regular DR drills
```

## 🔒 Security

### Least Privilege

```yaml
# RBAC
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```

### Secrets Management

```yaml
# Use external secret management
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: db-secret
spec:
  secretStoreRef:
    name: vault-backend
    kind: SecretStore
  target:
    name: db-secret
  data:
  - secretKey: password
    remoteRef:
      key: database/password
```

### Network Policies

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: restrict-traffic
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

### Regular Updates

```bash
# Update container images
kubectl set image deployment/myapp app=myapp:v2.0.0

# Update Kubernetes
kubectl upgrade

# Security patches
# Regular vulnerability scanning
# Automated updates for non-critical
```

## 📊 Configuration Management

### Environment Variables

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: "postgresql://db:5432/mydb"
  log_level: "info"
  max_connections: "100"

---
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  api_key: base64encoded
```

### Feature Flags

```python
# LaunchDarkly example
import ldclient
from ldclient.config import Config

ldclient.set_config(Config("your-sdk-key"))
client = ldclient.get()

# Check feature flag
if client.variation("new-feature", {"key": user_id}, False):
    new_feature()
else:
    old_feature()
```

## 📈 Performance Optimization

### Resource Optimization

```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "250m"
  limits:
    memory: "512Mi"
    cpu: "500m"
```

### Caching

```python
# Redis caching
import redis
cache = redis.Redis(host='redis', port=6379, db=0)

def get_data(key):
    cached = cache.get(key)
    if cached:
        return json.loads(cached)
    
    data = fetch_from_database()
    cache.setex(key, 3600, json.dumps(data))  # 1 hour TTL
    return data
```

### Database Optimization

```sql
-- Indexes
CREATE INDEX idx_user_email ON users(email);

-- Query optimization
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 123;

-- Connection pooling
-- Use connection poolers (PgBouncer, ProxySQL)
```

## 🚨 Incident Response

### Incident Lifecycle

```
Detection → Response → Mitigation → Resolution → Post-Mortem
```

### Runbooks

```markdown
# High CPU Usage

## Symptoms
- CPU usage > 90%
- Slow response times
- Timeout errors

## Investigation
1. Check top processes: `kubectl top pods`
2. Review application logs: `kubectl logs pod-name`
3. Check recent deployments: `kubectl rollout history`
4. Review metrics: Grafana dashboards

## Resolution
1. Scale up: `kubectl scale deployment myapp --replicas=10`
2. Restart problematic pods: `kubectl delete pod pod-name`
3. Check for infinite loops in code
4. Review resource limits

## Prevention
- Set up HPA
- Monitor resource usage
- Regular capacity planning
- Load testing
```

## ✅ Best Practices

### 1. Implement Proper Monitoring

```yaml
# Monitor:
# - Application metrics
# - Infrastructure metrics
# - Business metrics
# - SLOs
```

### 2. Use Deployment Strategies

```yaml
# Choose right strategy:
# - Blue-Green: Zero downtime, instant rollback
# - Canary: Gradual rollout, risk mitigation
# - Rolling: Standard, resource efficient
```

### 3. Plan for Failures

```yaml
# Assume failures will happen
# Design for resilience
# Implement circuit breakers
# Use retries with backoff
```

### 4. Regular Backups

```yaml
# Automated backups
# Test restore procedures
# Off-site backups
# Regular DR drills
```

### 5. Security First

```yaml
# Least privilege
# Secrets management
# Network policies
# Regular updates
# Security scanning
```

### 6. Document Everything

```yaml
# Architecture diagrams
# Runbooks
# Procedures
# Decisions (ADRs)
```

## ✅ Mastery Checklist

- [ ] Implement high availability
- [ ] Set up monitoring and alerting
- [ ] Create backup strategy
- [ ] Harden security
- [ ] Document procedures
- [ ] Create runbooks
- [ ] Plan disaster recovery
- [ ] Test recovery procedures
- [ ] Optimize performance
- [ ] Prepare incident response

---

**Next Steps**:
- Learn [On-Call & SRE](./oncall-sre.md)
- Explore [Troubleshooting Guide](./troubleshooting-guide.md)
- Master [Service Mesh](./service-mesh.md)

**Remember**: Production excellence requires careful planning, robust practices, and continuous improvement. Start with basics, implement best practices, and always learn from incidents. Good production practices prevent problems and enable fast recovery when issues occur.
