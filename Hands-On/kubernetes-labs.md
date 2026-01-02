# Kubernetes Labs: Complete Hands-On Training

## 🎯 Introduction

This comprehensive lab guide provides practical, hands-on exercises to master Kubernetes. Each lab includes objectives, prerequisites, step-by-step instructions, and verification steps.

### Prerequisites

- Docker installed
- kubectl installed
- Local Kubernetes cluster (minikube, kind, or k3d)
- Basic understanding of Kubernetes concepts

### Setting Up Your Lab Environment

```bash
# Option 1: Minikube
minikube start --memory=4096 --cpus=2 --driver=docker

# Option 2: Kind (Kubernetes in Docker)
kind create cluster --name k8s-labs

# Option 3: k3d (Lightweight k3s)
k3d cluster create k8s-labs

# Verify cluster is running
kubectl cluster-info
kubectl get nodes
```

---

## Lab 1: Pod Fundamentals

### Objective
Learn to create, manage, and troubleshoot Kubernetes Pods.

### Duration: 30 minutes

### Tasks

#### 1.1 Create a Simple Pod

```bash
# Create pod using imperative command
kubectl run nginx-pod --image=nginx:1.21 --port=80

# Verify pod is running
kubectl get pods
kubectl get pods -o wide

# View pod details
kubectl describe pod nginx-pod
```

#### 1.2 Create Pod from YAML Manifest

Create file `lab1-pod.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-pod
  labels:
    app: webapp
    tier: frontend
    environment: lab
spec:
  containers:
  - name: webapp
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
    env:
    - name: ENVIRONMENT
      value: "development"
    - name: LOG_LEVEL
      value: "debug"
```

```bash
# Apply the manifest
kubectl apply -f lab1-pod.yaml

# Verify
kubectl get pods -l app=webapp
kubectl describe pod webapp-pod
```

#### 1.3 Pod Interaction

```bash
# View pod logs
kubectl logs webapp-pod
kubectl logs webapp-pod -f  # Follow logs

# Execute commands in pod
kubectl exec webapp-pod -- ls /usr/share/nginx/html
kubectl exec -it webapp-pod -- /bin/bash

# Inside the container:
# cat /etc/nginx/nginx.conf
# curl localhost
# exit

# Copy files to/from pod
kubectl cp webapp-pod:/etc/nginx/nginx.conf ./nginx.conf
echo "Hello from Lab 1" > index.html
kubectl cp index.html webapp-pod:/usr/share/nginx/html/
```

#### 1.4 Multi-Container Pod (Sidecar Pattern)

Create file `lab1-multi-container.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
  # Main application container
  - name: app
    image: nginx:1.21
    ports:
    - containerPort: 80
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx
  
  # Sidecar container for log shipping
  - name: log-sidecar
    image: busybox:1.35
    command: ['sh', '-c', 'tail -F /var/log/nginx/access.log']
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx
  
  volumes:
  - name: shared-logs
    emptyDir: {}
```

```bash
kubectl apply -f lab1-multi-container.yaml

# View logs from specific container
kubectl logs multi-container-pod -c app
kubectl logs multi-container-pod -c log-sidecar

# Execute in specific container
kubectl exec -it multi-container-pod -c app -- /bin/bash
```

#### 1.5 Cleanup

```bash
kubectl delete pod nginx-pod webapp-pod multi-container-pod
```

### Verification Checklist
- [ ] Created pod imperatively
- [ ] Created pod from YAML
- [ ] Viewed pod logs
- [ ] Executed commands in pod
- [ ] Created multi-container pod
- [ ] Copied files to/from pod

---

## Lab 2: Deployments and ReplicaSets

### Objective
Master Deployment management including scaling, updates, and rollbacks.

### Duration: 45 minutes

### Tasks

#### 2.1 Create a Deployment

Create file `lab2-deployment.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-deployment
  labels:
    app: webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
        version: v1
    spec:
      containers:
      - name: webapp
        image: nginx:1.21
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "200m"
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 10
```

```bash
kubectl apply -f lab2-deployment.yaml

# Verify deployment
kubectl get deployments
kubectl get replicasets
kubectl get pods -l app=webapp

# View deployment details
kubectl describe deployment webapp-deployment
```

#### 2.2 Scaling

```bash
# Scale up
kubectl scale deployment webapp-deployment --replicas=5
kubectl get pods -l app=webapp -w  # Watch scaling

# Scale down
kubectl scale deployment webapp-deployment --replicas=2

# Autoscaling (requires metrics-server)
kubectl autoscale deployment webapp-deployment --min=2 --max=10 --cpu-percent=50
kubectl get hpa
```

#### 2.3 Rolling Updates

```bash
# Update image (triggers rolling update)
kubectl set image deployment/webapp-deployment webapp=nginx:1.22

# Watch the rollout
kubectl rollout status deployment/webapp-deployment

# View rollout history
kubectl rollout history deployment/webapp-deployment

# View specific revision
kubectl rollout history deployment/webapp-deployment --revision=2
```

#### 2.4 Rollbacks

```bash
# Rollback to previous version
kubectl rollout undo deployment/webapp-deployment

# Rollback to specific revision
kubectl rollout undo deployment/webapp-deployment --to-revision=1

# Verify
kubectl describe deployment webapp-deployment | grep Image
```

#### 2.5 Deployment Strategies

Create file `lab2-canary.yaml`:
```yaml
# Canary Deployment - Run both versions
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-canary
spec:
  replicas: 1  # Small percentage of traffic
  selector:
    matchLabels:
      app: webapp
      track: canary
  template:
    metadata:
      labels:
        app: webapp
        track: canary
        version: v2
    spec:
      containers:
      - name: webapp
        image: nginx:1.23
        ports:
        - containerPort: 80
```

```bash
kubectl apply -f lab2-canary.yaml

# Both deployments serve traffic through same service
kubectl get pods -l app=webapp --show-labels
```

#### 2.6 Cleanup

```bash
kubectl delete deployment webapp-deployment webapp-canary
kubectl delete hpa webapp-deployment
```

### Verification Checklist
- [ ] Created deployment with replicas
- [ ] Scaled deployment up and down
- [ ] Performed rolling update
- [ ] Rolled back deployment
- [ ] Understood deployment strategies

---

## Lab 3: Services and Networking

### Objective
Learn to expose applications using different Service types.

### Duration: 45 minutes

### Tasks

#### 3.1 Setup Deployment for Services

```bash
# Create deployment first
kubectl create deployment web --image=nginx:1.21 --replicas=3
kubectl get pods -l app=web
```

#### 3.2 ClusterIP Service (Internal)

Create file `lab3-clusterip.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-clusterip
spec:
  type: ClusterIP
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
```

```bash
kubectl apply -f lab3-clusterip.yaml

# Verify service
kubectl get svc web-clusterip
kubectl describe svc web-clusterip

# Test from within cluster
kubectl run test-pod --image=busybox:1.35 --rm -it --restart=Never -- wget -qO- http://web-clusterip

# Check endpoints
kubectl get endpoints web-clusterip
```

#### 3.3 NodePort Service (External via Node)

Create file `lab3-nodeport.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-nodeport
spec:
  type: NodePort
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080  # Optional: auto-assigned if not specified
    protocol: TCP
```

```bash
kubectl apply -f lab3-nodeport.yaml
kubectl get svc web-nodeport

# Get node IP
kubectl get nodes -o wide

# Access via NodePort (for minikube)
minikube service web-nodeport --url

# Or access directly: http://<node-ip>:30080
```

#### 3.4 LoadBalancer Service (Cloud)

Create file `lab3-loadbalancer.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
```

```bash
kubectl apply -f lab3-loadbalancer.yaml
kubectl get svc web-loadbalancer

# For minikube, run in separate terminal:
minikube tunnel

# Now check external IP
kubectl get svc web-loadbalancer
```

#### 3.5 Headless Service (StatefulSet Use Case)

Create file `lab3-headless.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-headless
spec:
  clusterIP: None  # Headless
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
```

```bash
kubectl apply -f lab3-headless.yaml

# DNS returns pod IPs directly
kubectl run test-pod --image=busybox:1.35 --rm -it --restart=Never -- nslookup web-headless
```

#### 3.6 Service Discovery via DNS

```bash
# Test DNS resolution
kubectl run dns-test --image=busybox:1.35 --rm -it --restart=Never -- sh

# Inside the pod:
# nslookup web-clusterip
# nslookup web-clusterip.default.svc.cluster.local
# wget -qO- http://web-clusterip
# exit
```

#### 3.7 Cleanup

```bash
kubectl delete deployment web
kubectl delete svc web-clusterip web-nodeport web-loadbalancer web-headless
```

### Verification Checklist
- [ ] Created ClusterIP service
- [ ] Tested internal service connectivity
- [ ] Created NodePort service
- [ ] Accessed service via NodePort
- [ ] Created LoadBalancer service
- [ ] Understood headless services

---

## Lab 4: ConfigMaps and Secrets

### Objective
Learn to manage application configuration and sensitive data.

### Duration: 30 minutes

### Tasks

#### 4.1 Create ConfigMap from Literal Values

```bash
# Imperative
kubectl create configmap app-config \
  --from-literal=DATABASE_HOST=mysql.default.svc \
  --from-literal=DATABASE_PORT=3306 \
  --from-literal=LOG_LEVEL=debug

kubectl get configmap app-config -o yaml
```

#### 4.2 Create ConfigMap from File

Create file `app.properties`:
```properties
# Application Configuration
database.host=mysql.default.svc
database.port=3306
database.name=myapp
cache.enabled=true
cache.ttl=3600
log.level=info
```

Create file `nginx.conf`:
```nginx
server {
    listen 80;
    server_name localhost;
    
    location / {
        root /usr/share/nginx/html;
        index index.html;
    }
    
    location /health {
        return 200 'healthy';
        add_header Content-Type text/plain;
    }
}
```

```bash
# Create from files
kubectl create configmap app-files \
  --from-file=app.properties \
  --from-file=nginx.conf

kubectl describe configmap app-files
```

#### 4.3 Create ConfigMap from YAML

Create file `lab4-configmap.yaml`:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: webapp-config
data:
  # Simple key-value
  DATABASE_URL: "postgresql://db:5432/myapp"
  REDIS_HOST: "redis.default.svc"
  
  # Multi-line configuration file
  app.yaml: |
    server:
      port: 8080
      host: 0.0.0.0
    database:
      pool_size: 10
      timeout: 30
    logging:
      level: info
      format: json
```

```bash
kubectl apply -f lab4-configmap.yaml
```

#### 4.4 Create Secrets

```bash
# Create secret from literal (base64 encoded automatically)
kubectl create secret generic db-credentials \
  --from-literal=username=admin \
  --from-literal=password='S3cr3tP@ssw0rd!'

# View secret (values are base64 encoded)
kubectl get secret db-credentials -o yaml

# Decode secret value
kubectl get secret db-credentials -o jsonpath='{.data.password}' | base64 -d
```

Create file `lab4-secret.yaml`:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
stringData:  # Use stringData for plain text (auto-encoded)
  api-key: "my-super-secret-api-key"
  jwt-secret: "jwt-signing-secret-key-12345"
data:  # Use data for pre-encoded values
  # echo -n 'admin' | base64
  db-username: YWRtaW4=
  # echo -n 'password123' | base64
  db-password: cGFzc3dvcmQxMjM=
```

```bash
kubectl apply -f lab4-secret.yaml
```

#### 4.5 Use ConfigMap and Secret in Pod

Create file `lab4-pod.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-demo-pod
spec:
  containers:
  - name: app
    image: nginx:1.21
    
    # Environment variables from ConfigMap
    env:
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: webapp-config
          key: DATABASE_URL
    
    # Environment variables from Secret
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: app-secrets
          key: db-username
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secrets
          key: db-password
    
    # All keys from ConfigMap as env vars
    envFrom:
    - configMapRef:
        name: app-config
    
    volumeMounts:
    # Mount ConfigMap as files
    - name: config-volume
      mountPath: /etc/config
    # Mount Secret as files
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true
  
  volumes:
  - name: config-volume
    configMap:
      name: webapp-config
  - name: secret-volume
    secret:
      secretName: app-secrets
```

```bash
kubectl apply -f lab4-pod.yaml

# Verify environment variables
kubectl exec config-demo-pod -- env | grep -E "DATABASE|DB_|LOG"

# Verify mounted files
kubectl exec config-demo-pod -- ls /etc/config
kubectl exec config-demo-pod -- cat /etc/config/app.yaml

kubectl exec config-demo-pod -- ls /etc/secrets
kubectl exec config-demo-pod -- cat /etc/secrets/api-key
```

#### 4.6 Cleanup

```bash
kubectl delete pod config-demo-pod
kubectl delete configmap app-config app-files webapp-config
kubectl delete secret db-credentials app-secrets
```

### Verification Checklist
- [ ] Created ConfigMap from literals
- [ ] Created ConfigMap from files
- [ ] Created Secret
- [ ] Used ConfigMap as environment variables
- [ ] Used Secret as environment variables
- [ ] Mounted ConfigMap as volume
- [ ] Mounted Secret as volume

---

## Lab 5: Persistent Volumes and Claims

### Objective
Learn to manage persistent storage for stateful applications.

### Duration: 45 minutes

### Tasks

#### 5.1 Create PersistentVolume (Local)

Create file `lab5-pv.yaml`:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /tmp/k8s-lab-data
```

```bash
kubectl apply -f lab5-pv.yaml
kubectl get pv
kubectl describe pv local-pv
```

#### 5.2 Create PersistentVolumeClaim

Create file `lab5-pvc.yaml`:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-pvc
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

```bash
kubectl apply -f lab5-pvc.yaml
kubectl get pvc
kubectl describe pvc local-pvc

# Check PV is now bound
kubectl get pv
```

#### 5.3 Use PVC in Pod

Create file `lab5-pod.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: storage-pod
spec:
  containers:
  - name: app
    image: nginx:1.21
    volumeMounts:
    - name: data
      mountPath: /usr/share/nginx/html
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: local-pvc
```

```bash
kubectl apply -f lab5-pod.yaml

# Write data to persistent storage
kubectl exec storage-pod -- sh -c 'echo "Persistent Data!" > /usr/share/nginx/html/index.html'

# Verify data
kubectl exec storage-pod -- cat /usr/share/nginx/html/index.html

# Delete pod
kubectl delete pod storage-pod

# Recreate pod
kubectl apply -f lab5-pod.yaml

# Data persists!
kubectl exec storage-pod -- cat /usr/share/nginx/html/index.html
```

#### 5.4 Dynamic Provisioning with StorageClass

```bash
# View available storage classes
kubectl get storageclass

# For minikube, standard storage class is available
kubectl describe storageclass standard
```

Create file `lab5-dynamic-pvc.yaml`:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard  # Use default StorageClass
  resources:
    requests:
      storage: 1Gi
```

```bash
kubectl apply -f lab5-dynamic-pvc.yaml
kubectl get pvc dynamic-pvc
kubectl get pv  # PV created automatically
```

#### 5.5 StatefulSet with Persistent Storage

Create file `lab5-statefulset.yaml`:
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: web-headless
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        volumeMounts:
        - name: data
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: standard
      resources:
        requests:
          storage: 100Mi
---
apiVersion: v1
kind: Service
metadata:
  name: web-headless
spec:
  clusterIP: None
  selector:
    app: web
  ports:
  - port: 80
```

```bash
kubectl apply -f lab5-statefulset.yaml

# Watch pods being created in order
kubectl get pods -l app=web -w

# Each pod gets its own PVC
kubectl get pvc

# Write unique data to each pod
kubectl exec web-0 -- sh -c 'echo "web-0 data" > /usr/share/nginx/html/index.html'
kubectl exec web-1 -- sh -c 'echo "web-1 data" > /usr/share/nginx/html/index.html'
kubectl exec web-2 -- sh -c 'echo "web-2 data" > /usr/share/nginx/html/index.html'

# Verify each pod has unique data
kubectl exec web-0 -- cat /usr/share/nginx/html/index.html
kubectl exec web-1 -- cat /usr/share/nginx/html/index.html
kubectl exec web-2 -- cat /usr/share/nginx/html/index.html
```

#### 5.6 Cleanup

```bash
kubectl delete statefulset web
kubectl delete svc web-headless
kubectl delete pvc --all
kubectl delete pv local-pv
kubectl delete pod storage-pod
```

### Verification Checklist
- [ ] Created PersistentVolume
- [ ] Created PersistentVolumeClaim
- [ ] Used PVC in Pod
- [ ] Verified data persistence
- [ ] Used dynamic provisioning
- [ ] Deployed StatefulSet with persistent storage

---

## Lab 6: Network Policies

### Objective
Implement network segmentation and security using Network Policies.

### Duration: 45 minutes

### Tasks

#### 6.1 Setup Test Environment

Create file `lab6-setup.yaml`:
```yaml
# Frontend namespace
apiVersion: v1
kind: Namespace
metadata:
  name: frontend
  labels:
    purpose: frontend
---
# Backend namespace
apiVersion: v1
kind: Namespace
metadata:
  name: backend
  labels:
    purpose: backend
---
# Frontend deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  namespace: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
        role: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
---
# Backend API deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
  namespace: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
        role: backend
    spec:
      containers:
      - name: api
        image: nginx:1.21
        ports:
        - containerPort: 80
---
# Backend database deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: database
  namespace: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: database
  template:
    metadata:
      labels:
        app: database
        role: database
    spec:
      containers:
      - name: db
        image: nginx:1.21
        ports:
        - containerPort: 80
---
# Services
apiVersion: v1
kind: Service
metadata:
  name: web
  namespace: frontend
spec:
  selector:
    app: web
  ports:
  - port: 80
---
apiVersion: v1
kind: Service
metadata:
  name: api
  namespace: backend
spec:
  selector:
    app: api
  ports:
  - port: 80
---
apiVersion: v1
kind: Service
metadata:
  name: database
  namespace: backend
spec:
  selector:
    app: database
  ports:
  - port: 80
```

```bash
kubectl apply -f lab6-setup.yaml

# Verify setup
kubectl get pods -n frontend
kubectl get pods -n backend
kubectl get svc -n frontend
kubectl get svc -n backend
```

#### 6.2 Test Default Connectivity (No Policies)

```bash
# Test frontend → backend API (should work)
kubectl exec -n frontend deploy/web -- wget -qO- --timeout=2 http://api.backend.svc.cluster.local

# Test frontend → backend database (should work - we'll block this)
kubectl exec -n frontend deploy/web -- wget -qO- --timeout=2 http://database.backend.svc.cluster.local

# Test backend API → database (should work)
kubectl exec -n backend deploy/api -- wget -qO- --timeout=2 http://database.backend.svc.cluster.local
```

#### 6.3 Default Deny Policy

Create file `lab6-deny-all.yaml`:
```yaml
# Deny all ingress to backend namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: backend
spec:
  podSelector: {}  # Applies to all pods in namespace
  policyTypes:
  - Ingress
```

```bash
kubectl apply -f lab6-deny-all.yaml

# Now all traffic to backend is blocked
kubectl exec -n frontend deploy/web -- wget -qO- --timeout=2 http://api.backend.svc.cluster.local
# Should timeout
```

#### 6.4 Allow Specific Traffic

Create file `lab6-allow-frontend-to-api.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-api
  namespace: backend
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          purpose: frontend
      podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 80
```

Create file `lab6-allow-api-to-db.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-api-to-database
  namespace: backend
spec:
  podSelector:
    matchLabels:
      app: database
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: api
    ports:
    - protocol: TCP
      port: 80
```

```bash
kubectl apply -f lab6-allow-frontend-to-api.yaml
kubectl apply -f lab6-allow-api-to-db.yaml

# Test frontend → API (should work now)
kubectl exec -n frontend deploy/web -- wget -qO- --timeout=2 http://api.backend.svc.cluster.local

# Test frontend → database (should still be blocked)
kubectl exec -n frontend deploy/web -- wget -qO- --timeout=2 http://database.backend.svc.cluster.local
# Should timeout

# Test API → database (should work)
kubectl exec -n backend deploy/api -- wget -qO- --timeout=2 http://database.backend.svc.cluster.local
```

#### 6.5 Egress Policy

Create file `lab6-egress.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-egress
  namespace: backend
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Egress
  egress:
  # Allow DNS
  - to: []
    ports:
    - protocol: UDP
      port: 53
  # Allow database
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 80
  # Allow external HTTPS
  - to: []
    ports:
    - protocol: TCP
      port: 443
```

```bash
kubectl apply -f lab6-egress.yaml

# Verify egress rules
kubectl describe networkpolicy api-egress -n backend
```

#### 6.6 Cleanup

```bash
kubectl delete namespace frontend backend
```

### Verification Checklist
- [ ] Created multi-namespace environment
- [ ] Tested default connectivity
- [ ] Applied default deny policy
- [ ] Created specific allow rules
- [ ] Tested network isolation
- [ ] Applied egress policies

---

## Lab 7: RBAC (Role-Based Access Control)

### Objective
Implement fine-grained access control using RBAC.

### Duration: 45 minutes

### Tasks

#### 7.1 Create ServiceAccount

Create file `lab7-serviceaccount.yaml`:
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: developer
  namespace: default
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: viewer
  namespace: default
```

```bash
kubectl apply -f lab7-serviceaccount.yaml
kubectl get serviceaccounts
```

#### 7.2 Create Role (Namespace-scoped)

Create file `lab7-role.yaml`:
```yaml
# Developer role - can manage pods and deployments
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log", "pods/exec"]
  verbs: ["get", "list", "watch", "create", "delete"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: [""]
  resources: ["services"]
  verbs: ["get", "list", "watch", "create"]
- apiGroups: [""]
  resources: ["configmaps", "secrets"]
  verbs: ["get", "list"]
---
# Viewer role - read-only access
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: viewer
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods", "services", "configmaps"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch"]
```

```bash
kubectl apply -f lab7-role.yaml
kubectl get roles
kubectl describe role developer
```

#### 7.3 Create RoleBinding

Create file `lab7-rolebinding.yaml`:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer-binding
  namespace: default
subjects:
- kind: ServiceAccount
  name: developer
  namespace: default
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: viewer-binding
  namespace: default
subjects:
- kind: ServiceAccount
  name: viewer
  namespace: default
roleRef:
  kind: Role
  name: viewer
  apiGroup: rbac.authorization.k8s.io
```

```bash
kubectl apply -f lab7-rolebinding.yaml
kubectl get rolebindings
```

#### 7.4 Test RBAC Permissions

```bash
# Test as developer
kubectl auth can-i create pods --as=system:serviceaccount:default:developer
# yes

kubectl auth can-i delete deployments --as=system:serviceaccount:default:developer
# yes

kubectl auth can-i create secrets --as=system:serviceaccount:default:developer
# no

# Test as viewer
kubectl auth can-i get pods --as=system:serviceaccount:default:viewer
# yes

kubectl auth can-i delete pods --as=system:serviceaccount:default:viewer
# no

kubectl auth can-i create deployments --as=system:serviceaccount:default:viewer
# no
```

#### 7.5 Create ClusterRole (Cluster-wide)

Create file `lab7-clusterrole.yaml`:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: namespace-admin
rules:
- apiGroups: [""]
  resources: ["namespaces"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["metrics.k8s.io"]
  resources: ["pods", "nodes"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: developer-cluster-binding
subjects:
- kind: ServiceAccount
  name: developer
  namespace: default
roleRef:
  kind: ClusterRole
  name: namespace-admin
  apiGroup: rbac.authorization.k8s.io
```

```bash
kubectl apply -f lab7-clusterrole.yaml

# Developer can now list namespaces
kubectl auth can-i list namespaces --as=system:serviceaccount:default:developer
# yes
```

#### 7.6 Pod with ServiceAccount

Create file `lab7-pod.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: developer-pod
spec:
  serviceAccountName: developer
  containers:
  - name: kubectl
    image: bitnami/kubectl:latest
    command: ["sleep", "infinity"]
```

```bash
kubectl apply -f lab7-pod.yaml

# Test from inside the pod
kubectl exec -it developer-pod -- kubectl get pods
kubectl exec -it developer-pod -- kubectl get secrets
# Should show permissions based on developer role
```

#### 7.7 Cleanup

```bash
kubectl delete pod developer-pod
kubectl delete rolebinding developer-binding viewer-binding
kubectl delete clusterrolebinding developer-cluster-binding
kubectl delete role developer viewer
kubectl delete clusterrole namespace-admin
kubectl delete serviceaccount developer viewer
```

### Verification Checklist
- [ ] Created ServiceAccounts
- [ ] Created Roles with specific permissions
- [ ] Created RoleBindings
- [ ] Tested RBAC permissions with `can-i`
- [ ] Created ClusterRole and ClusterRoleBinding
- [ ] Used ServiceAccount in Pod

---

## Lab 8: Ingress Controllers

### Objective
Expose multiple services through a single entry point with Ingress.

### Duration: 45 minutes

### Tasks

#### 8.1 Install Ingress Controller

```bash
# For minikube
minikube addons enable ingress
kubectl get pods -n ingress-nginx

# For kind, create cluster with ingress config
# kind create cluster --config kind-ingress-config.yaml

# Verify ingress controller
kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx
```

#### 8.2 Deploy Sample Applications

Create file `lab8-apps.yaml`:
```yaml
# App 1: Web Frontend
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-frontend
  template:
    metadata:
      labels:
        app: web-frontend
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html
      initContainers:
      - name: init
        image: busybox
        command: ['sh', '-c', 'echo "<h1>Frontend App</h1>" > /html/index.html']
        volumeMounts:
        - name: html
          mountPath: /html
      volumes:
      - name: html
        emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: web-frontend
spec:
  selector:
    app: web-frontend
  ports:
  - port: 80
---
# App 2: API Backend
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: api-backend
  template:
    metadata:
      labels:
        app: api-backend
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html
      initContainers:
      - name: init
        image: busybox
        command: ['sh', '-c', 'echo "{\"message\": \"API Response\"}" > /html/index.html']
        volumeMounts:
        - name: html
          mountPath: /html
      volumes:
      - name: html
        emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: api-backend
spec:
  selector:
    app: api-backend
  ports:
  - port: 80
```

```bash
kubectl apply -f lab8-apps.yaml
kubectl get pods
kubectl get svc
```

#### 8.3 Path-Based Routing

Create file `lab8-ingress-path.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: path-based-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: myapp.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-frontend
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-backend
            port:
              number: 80
```

```bash
kubectl apply -f lab8-ingress-path.yaml
kubectl get ingress
kubectl describe ingress path-based-ingress

# Get ingress IP
kubectl get ingress path-based-ingress -o jsonpath='{.status.loadBalancer.ingress[0].ip}'

# For minikube
minikube ip

# Add to /etc/hosts (or C:\Windows\System32\drivers\etc\hosts)
# <minikube-ip> myapp.local

# Test
curl http://myapp.local/
curl http://myapp.local/api
```

#### 8.4 Host-Based Routing

Create file `lab8-ingress-host.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: host-based-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: frontend.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-frontend
            port:
              number: 80
  - host: api.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-backend
            port:
              number: 80
```

```bash
kubectl apply -f lab8-ingress-host.yaml

# Add to /etc/hosts
# <minikube-ip> frontend.local api.local

# Test
curl http://frontend.local/
curl http://api.local/
```

#### 8.5 TLS Termination

```bash
# Generate self-signed certificate
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt \
  -subj "/CN=myapp.local"

# Create TLS secret
kubectl create secret tls myapp-tls --cert=tls.crt --key=tls.key
```

Create file `lab8-ingress-tls.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - myapp.local
    secretName: myapp-tls
  rules:
  - host: myapp.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-frontend
            port:
              number: 80
```

```bash
kubectl apply -f lab8-ingress-tls.yaml

# Test HTTPS
curl -k https://myapp.local/
```

#### 8.6 Cleanup

```bash
kubectl delete ingress --all
kubectl delete deployment web-frontend api-backend
kubectl delete svc web-frontend api-backend
kubectl delete secret myapp-tls
rm tls.key tls.crt
```

### Verification Checklist
- [ ] Installed Ingress controller
- [ ] Deployed sample applications
- [ ] Created path-based routing
- [ ] Created host-based routing
- [ ] Configured TLS termination
- [ ] Tested all routing rules

---

## Lab 9: Horizontal Pod Autoscaler (HPA)

### Objective
Implement automatic scaling based on resource utilization.

### Duration: 30 minutes

### Tasks

#### 9.1 Install Metrics Server

```bash
# Check if metrics-server is installed
kubectl get deployment metrics-server -n kube-system

# For minikube
minikube addons enable metrics-server

# Verify
kubectl top nodes
kubectl top pods
```

#### 9.2 Deploy Application with Resource Requests

Create file `lab9-deployment.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  replicas: 1
  selector:
    matchLabels:
      app: php-apache
  template:
    metadata:
      labels:
        app: php-apache
    spec:
      containers:
      - name: php-apache
        image: registry.k8s.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 200m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 256Mi
---
apiVersion: v1
kind: Service
metadata:
  name: php-apache
spec:
  selector:
    app: php-apache
  ports:
  - port: 80
    targetPort: 80
```

```bash
kubectl apply -f lab9-deployment.yaml
kubectl get pods
kubectl top pods
```

#### 9.3 Create HPA

```bash
# Imperative way
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10

# View HPA
kubectl get hpa
kubectl describe hpa php-apache
```

Or declarative:

Create file `lab9-hpa.yaml`:
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 70
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 60
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
```

```bash
kubectl apply -f lab9-hpa.yaml
```

#### 9.4 Generate Load

```bash
# Terminal 1: Watch HPA
kubectl get hpa -w

# Terminal 2: Watch Pods
kubectl get pods -l app=php-apache -w

# Terminal 3: Generate load
kubectl run -i --tty load-generator --rm --image=busybox:1.35 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"

# Wait and watch pods scale up
# Then stop the load generator (Ctrl+C) and watch scale down
```

#### 9.5 Cleanup

```bash
kubectl delete hpa php-apache php-apache-hpa
kubectl delete deployment php-apache
kubectl delete svc php-apache
```

### Verification Checklist
- [ ] Installed metrics-server
- [ ] Created deployment with resource requests
- [ ] Created HPA
- [ ] Generated load and observed scale-up
- [ ] Observed scale-down after load stopped

---

## Lab 10: Helm Package Manager

### Objective
Learn to manage Kubernetes applications with Helm charts.

### Duration: 45 minutes

### Tasks

#### 10.1 Install Helm

```bash
# Install Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Verify
helm version
```

#### 10.2 Add Helm Repositories

```bash
# Add popular repositories
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

# Update repositories
helm repo update

# Search for charts
helm search repo nginx
helm search repo prometheus
```

#### 10.3 Install Chart

```bash
# Install nginx
helm install my-nginx bitnami/nginx

# Check installation
helm list
kubectl get all -l app.kubernetes.io/instance=my-nginx

# Get values used
helm get values my-nginx

# Get all resources
helm get manifest my-nginx
```

#### 10.4 Customize Installation

```bash
# View available values
helm show values bitnami/nginx > nginx-values.yaml

# Install with custom values
helm install my-custom-nginx bitnami/nginx \
  --set replicaCount=3 \
  --set service.type=NodePort \
  --set service.nodePorts.http=30080

# Or with values file
helm install my-custom-nginx bitnami/nginx -f custom-values.yaml
```

#### 10.5 Create Custom Chart

```bash
# Create new chart
helm create mychart
cd mychart

# Chart structure
tree .
# mychart/
# ├── Chart.yaml
# ├── values.yaml
# ├── templates/
# │   ├── deployment.yaml
# │   ├── service.yaml
# │   ├── ingress.yaml
# │   └── ...
# └── charts/
```

Edit `values.yaml`:
```yaml
replicaCount: 2

image:
  repository: nginx
  tag: "1.21"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 50m
    memory: 64Mi
```

```bash
# Lint chart
helm lint .

# Dry run
helm install --dry-run --debug myrelease .

# Install
helm install myrelease .

# Verify
kubectl get all -l app.kubernetes.io/instance=myrelease
```

#### 10.6 Upgrade and Rollback

```bash
# Upgrade
helm upgrade my-nginx bitnami/nginx --set replicaCount=5

# View history
helm history my-nginx

# Rollback
helm rollback my-nginx 1

# Uninstall
helm uninstall my-nginx
```

#### 10.7 Cleanup

```bash
helm uninstall my-nginx my-custom-nginx myrelease
cd ..
rm -rf mychart
```

### Verification Checklist
- [ ] Installed Helm
- [ ] Added repositories
- [ ] Installed chart from repository
- [ ] Customized chart installation
- [ ] Created custom chart
- [ ] Performed upgrade and rollback

---

## Lab 11: Pod Disruption Budgets

### Objective
Ensure application availability during voluntary disruptions.

### Duration: 20 minutes

### Tasks

#### 11.1 Create Deployment

Create file `lab11-deployment.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 5
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
```

```bash
kubectl apply -f lab11-deployment.yaml
kubectl get pods -l app=webapp
```

#### 11.2 Create PodDisruptionBudget

Create file `lab11-pdb.yaml`:
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: webapp-pdb
spec:
  minAvailable: 3  # At least 3 pods must be available
  # OR use maxUnavailable: 2  # At most 2 pods can be unavailable
  selector:
    matchLabels:
      app: webapp
```

```bash
kubectl apply -f lab11-pdb.yaml
kubectl get pdb
kubectl describe pdb webapp-pdb
```

#### 11.3 Test PDB

```bash
# Try to drain a node (will respect PDB)
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data

# The drain will pause if it would violate the PDB
# Uncordon the node
kubectl uncordon <node-name>
```

#### 11.4 Cleanup

```bash
kubectl delete pdb webapp-pdb
kubectl delete deployment webapp
```

### Verification Checklist
- [ ] Created deployment with multiple replicas
- [ ] Created PodDisruptionBudget
- [ ] Understood minAvailable vs maxUnavailable
- [ ] Tested PDB behavior during drain

---

## Lab 12: Resource Quotas and Limits

### Objective
Implement resource management at namespace level.

### Duration: 30 minutes

### Tasks

#### 12.1 Create Namespace with Quota

Create file `lab12-quota.yaml`:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: quota-demo
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: quota-demo
spec:
  hard:
    requests.cpu: "2"
    requests.memory: 2Gi
    limits.cpu: "4"
    limits.memory: 4Gi
    pods: "10"
    services: "5"
    secrets: "10"
    configmaps: "10"
    persistentvolumeclaims: "5"
---
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: quota-demo
spec:
  limits:
  - default:
      cpu: 500m
      memory: 256Mi
    defaultRequest:
      cpu: 100m
      memory: 128Mi
    max:
      cpu: "1"
      memory: 1Gi
    min:
      cpu: 50m
      memory: 64Mi
    type: Container
```

```bash
kubectl apply -f lab12-quota.yaml
kubectl get resourcequota -n quota-demo
kubectl describe resourcequota compute-quota -n quota-demo
kubectl describe limitrange default-limits -n quota-demo
```

#### 12.2 Test Quota Enforcement

```bash
# Create pod without resources (LimitRange applies defaults)
kubectl run test-pod --image=nginx -n quota-demo
kubectl describe pod test-pod -n quota-demo | grep -A5 Limits

# Check quota usage
kubectl describe resourcequota compute-quota -n quota-demo

# Try to exceed quota
kubectl create deployment big-app --image=nginx --replicas=15 -n quota-demo
# Will fail due to quota
```

#### 12.3 Cleanup

```bash
kubectl delete namespace quota-demo
```

### Verification Checklist
- [ ] Created ResourceQuota
- [ ] Created LimitRange
- [ ] Tested default limits application
- [ ] Tested quota enforcement

---

## 🎯 Advanced Lab Challenges

### Challenge 1: Blue-Green Deployment
Implement a blue-green deployment strategy for zero-downtime updates.

### Challenge 2: Canary Deployment with Ingress
Use Ingress annotations to split traffic between stable and canary versions.

### Challenge 3: Custom Metrics HPA
Configure HPA to scale based on custom application metrics using Prometheus.

### Challenge 4: Multi-Cluster Service Discovery
Set up service discovery across multiple Kubernetes clusters.

### Challenge 5: GitOps with Argo CD
Implement GitOps workflow for continuous deployment.

---

## ✅ Lab Completion Checklist

- [ ] Lab 1: Pod Fundamentals
- [ ] Lab 2: Deployments and ReplicaSets
- [ ] Lab 3: Services and Networking
- [ ] Lab 4: ConfigMaps and Secrets
- [ ] Lab 5: Persistent Volumes
- [ ] Lab 6: Network Policies
- [ ] Lab 7: RBAC
- [ ] Lab 8: Ingress Controllers
- [ ] Lab 9: Horizontal Pod Autoscaler
- [ ] Lab 10: Helm Package Manager
- [ ] Lab 11: Pod Disruption Budgets
- [ ] Lab 12: Resource Quotas and Limits

---

## 📚 Next Steps

After completing these labs:
1. Practice CKA/CKAD exam scenarios
2. Build real applications on Kubernetes
3. Explore service mesh (Istio)
4. Learn GitOps with Argo CD
5. Study Kubernetes security in depth

**Remember**: Hands-on practice is the key to Kubernetes mastery. Repeat these labs until you can complete them without reference!

