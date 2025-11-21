# Kubernetes Hands-On Labs: Complete Practical Guide

## 🎯 Introduction

This comprehensive guide provides hands-on Kubernetes labs to master container orchestration. Each lab builds on previous knowledge.

## 📚 Lab Structure

Each lab includes:
- **Objective**: Learning goals
- **Prerequisites**: Required setup
- **Tasks**: Step-by-step exercises
- **YAML Examples**: Complete manifests
- **Commands**: kubectl commands
- **Verification**: Success criteria
- **Troubleshooting**: Common issues

## ☸️ Lab 1: Pod Fundamentals

### Objective
Understand pods, the smallest Kubernetes unit.

### Prerequisites
- Kubernetes cluster
- kubectl configured

### Tasks

#### Task 1: Create Basic Pod
```yaml
# pod-basic.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
    env: development
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
      name: http
    resources:
      requests:
        memory: "64Mi"
        cpu: "100m"
      limits:
        memory: "128Mi"
        cpu: "200m"
```

```bash
# Create pod
kubectl apply -f pod-basic.yaml

# View pod
kubectl get pods
kubectl get pods -o wide
kubectl get pods -l app=nginx

# Describe pod
kubectl describe pod nginx-pod

# View logs
kubectl logs nginx-pod
kubectl logs -f nginx-pod

# Execute commands
kubectl exec -it nginx-pod -- sh
kubectl exec nginx-pod -- nginx -v

# Port forward
kubectl port-forward nginx-pod 8080:80

# Delete pod
kubectl delete pod nginx-pod
```

#### Task 2: Multi-Container Pod
```yaml
# pod-multi-container.yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html
  
  - name: sidecar
    image: busybox
    command: ['sh', '-c', 'while true; do echo "$(date) - Sidecar running" >> /shared/log.txt; sleep 5; done']
    volumeMounts:
    - name: shared-data
      mountPath: /shared
  
  volumes:
  - name: shared-data
    emptyDir: {}
```

```bash
# Create pod
kubectl apply -f pod-multi-container.yaml

# View logs from specific container
kubectl logs multi-container-pod -c nginx
kubectl logs multi-container-pod -c sidecar

# Execute in specific container
kubectl exec -it multi-container-pod -c sidecar -- sh
```

#### Task 3: Pod with Health Checks
```yaml
# pod-healthchecks.yaml
apiVersion: v1
kind: Pod
metadata:
  name: healthy-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
    
    # Startup probe
    startupProbe:
      httpGet:
        path: /
        port: 80
      failureThreshold: 30
      periodSeconds: 10
    
    # Readiness probe
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
    
    # Liveness probe
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 30
      periodSeconds: 10
      failureThreshold: 3
```

```bash
# Create pod
kubectl apply -f pod-healthchecks.yaml

# Monitor pod status
kubectl get pods -w

# Check events
kubectl get events --sort-by=.metadata.creationTimestamp
```

### Verification
```bash
# Verify pod running
kubectl get pods -o wide

# Test connectivity
kubectl port-forward nginx-pod 8080:80
curl http://localhost:8080
```

## 🚀 Lab 2: Deployments and Scaling

### Objective
Master deployments, scaling, and updates.

### Tasks

#### Task 1: Create Deployment
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: app
        image: nginx:1.21
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
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

```bash
# Create deployment
kubectl apply -f deployment.yaml

# View deployment
kubectl get deployments
kubectl describe deployment web-app

# View ReplicaSet
kubectl get replicasets
kubectl describe replicaset web-app-xxxxx

# View pods
kubectl get pods -l app=web-app
```

#### Task 2: Scaling
```bash
# Manual scaling
kubectl scale deployment web-app --replicas=5

# Scale with file
kubectl scale --replicas=3 -f deployment.yaml

# View scaling events
kubectl get events --field-selector involvedObject.name=web-app
```

#### Task 3: Rolling Updates
```bash
# Update image
kubectl set image deployment/web-app app=nginx:1.22

# View rollout status
kubectl rollout status deployment/web-app

# View rollout history
kubectl rollout history deployment/web-app

# View specific revision
kubectl rollout history deployment/web-app --revision=2

# Rollback
kubectl rollout undo deployment/web-app
kubectl rollout undo deployment/web-app --to-revision=2

# Pause rollout
kubectl rollout pause deployment/web-app

# Resume rollout
kubectl rollout resume deployment/web-app

# Restart deployment
kubectl rollout restart deployment/web-app
```

#### Task 4: Horizontal Pod Autoscaler
```yaml
# hpa.yaml
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
```

```bash
# Create HPA
kubectl apply -f hpa.yaml

# View HPA
kubectl get hpa
kubectl describe hpa web-app-hpa

# Generate load (in another terminal)
kubectl run -it --rm load-generator --image=busybox --restart=Never -- /bin/sh
# Inside container: while true; do wget -q -O- http://web-app-service; done
```

### Verification
```bash
# Verify deployment
kubectl get deployment web-app
kubectl get pods -l app=web-app

# Test scaling
kubectl get hpa web-app-hpa
```

## 🌐 Lab 3: Services and Networking

### Objective
Understand services, networking, and service discovery.

### Tasks

#### Task 1: ClusterIP Service
```yaml
# service-clusterip.yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-service
spec:
  type: ClusterIP
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
```

```bash
# Create service
kubectl apply -f service-clusterip.yaml

# View service
kubectl get svc
kubectl describe svc web-app-service

# View endpoints
kubectl get endpoints web-app-service

# Test from pod
kubectl run -it --rm test --image=busybox --restart=Never -- wget -O- http://web-app-service:80
```

#### Task 2: NodePort Service
```yaml
# service-nodeport.yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-nodeport
spec:
  type: NodePort
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
    protocol: TCP
```

```bash
# Create service
kubectl apply -f service-nodeport.yaml

# Get node IP
kubectl get nodes -o wide

# Test from outside
curl http://<node-ip>:30080
```

#### Task 3: LoadBalancer Service
```yaml
# service-loadbalancer.yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-lb
spec:
  type: LoadBalancer
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
```

```bash
# Create service
kubectl apply -f service-loadbalancer.yaml

# View service (get external IP)
kubectl get svc web-app-lb

# Test
curl http://<external-ip>
```

#### Task 4: Ingress
```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-app-service
            port:
              number: 80
```

```bash
# Install ingress controller (NGINX)
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml

# Create ingress
kubectl apply -f ingress.yaml

# View ingress
kubectl get ingress
kubectl describe ingress web-app-ingress

# Test (add to /etc/hosts)
echo "<ingress-ip> app.example.com" >> /etc/hosts
curl http://app.example.com
```

### Verification
```bash
# Verify services
kubectl get svc
kubectl get endpoints

# Test connectivity
kubectl run -it --rm test --image=busybox --restart=Never -- wget -O- http://web-app-service
```

## 📦 Lab 4: ConfigMaps and Secrets

### Objective
Manage configuration and secrets.

### Tasks

#### Task 1: ConfigMap
```bash
# Create from literal
kubectl create configmap app-config \
  --from-literal=database_url=postgresql://db:5432/mydb \
  --from-literal=log_level=info

# Create from file
kubectl create configmap app-config-file \
  --from-file=config.yaml

# Create from YAML
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config-yaml
data:
  database_url: "postgresql://db:5432/mydb"
  log_level: "info"
  config.yaml: |
    server:
      port: 8080
      host: 0.0.0.0
EOF

# View ConfigMap
kubectl get configmap
kubectl get configmap app-config -o yaml
kubectl describe configmap app-config
```

#### Task 2: Secrets
```bash
# Create from literal
kubectl create secret generic app-secret \
  --from-literal=password=secret123 \
  --from-literal=api_key=key123

# Create from file
kubectl create secret generic app-secret-file \
  --from-file=password.txt

# Create from YAML
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: app-secret-yaml
type: Opaque
data:
  password: c2VjcmV0MTIz  # base64 encoded
  api_key: a2V5MTIz        # base64 encoded
stringData:
  username: admin
EOF

# View secret
kubectl get secret
kubectl get secret app-secret -o yaml
kubectl describe secret app-secret

# Decode secret
kubectl get secret app-secret -o jsonpath='{.data.password}' | base64 -d
```

#### Task 3: Use in Pods
```yaml
# pod-with-config.yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: nginx:1.21
    env:
    # From ConfigMap
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database_url
    - name: LOG_LEVEL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: log_level
    # From Secret
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: password
    # All from ConfigMap
    envFrom:
    - configMapRef:
        name: app-config
    # Volume mount
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
      readOnly: true
  volumes:
  - name: config-volume
    configMap:
      name: app-config-yaml
```

```bash
# Create pod
kubectl apply -f pod-with-config.yaml

# Verify
kubectl exec app-pod -- env | grep -E "DATABASE_URL|LOG_LEVEL|PASSWORD"
kubectl exec app-pod -- cat /etc/config/config.yaml
```

### Verification
```bash
# Verify ConfigMaps and Secrets
kubectl get configmap
kubectl get secret

# Verify in pod
kubectl exec app-pod -- env
```

## 💾 Lab 5: Persistent Storage

### Objective
Understand persistent volumes and storage.

### Tasks

#### Task 1: PersistentVolume and PersistentVolumeClaim
```yaml
# pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-volume
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /mnt/data
---
# pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: manual
```

```bash
# Create PV and PVC
kubectl apply -f pv.yaml
kubectl apply -f pvc.yaml

# View
kubectl get pv
kubectl get pvc
kubectl describe pv pv-volume
kubectl describe pvc pv-claim
```

#### Task 2: Use PVC in Pod
```yaml
# pod-with-pvc.yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-storage
spec:
  containers:
  - name: app
    image: nginx:1.21
    volumeMounts:
    - name: storage
      mountPath: /data
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: pv-claim
```

```bash
# Create pod
kubectl apply -f pod-with-pvc.yaml

# Write data
kubectl exec app-with-storage -- sh -c "echo 'Hello World' > /data/test.txt"

# Delete pod
kubectl delete pod app-with-storage

# Recreate pod (data persists)
kubectl apply -f pod-with-pvc.yaml

# Verify data
kubectl exec app-with-storage -- cat /data/test.txt
```

#### Task 3: StorageClass
```yaml
# storageclass.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
```

```bash
# Create StorageClass
kubectl apply -f storageclass.yaml

# View
kubectl get storageclass
```

### Verification
```bash
# Verify storage
kubectl get pv,pvc
kubectl describe pvc pv-claim

# Verify data persistence
kubectl exec app-with-storage -- ls -la /data
```

## ✅ Completion Checklist

- [ ] Created all resources
- [ ] Tested functionality
- [ ] Troubleshot issues
- [ ] Documented learnings
- [ ] Completed challenges

---

**Next**: Complete advanced labs for production scenarios.
