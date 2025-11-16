# Kubernetes: Complete Introduction & Fundamentals

## 🎯 What is Kubernetes?

Kubernetes (K8s) is an open-source container orchestration platform originally developed by Google, based on their internal Borg system. It automates deployment, scaling, and management of containerized applications across clusters of machines.

### Why Kubernetes?

**Before Kubernetes:**
- Manual container deployment
- Manual scaling
- Manual health checks
- Manual load balancing
- Difficult service discovery
- Complex networking

**With Kubernetes:**
- Automated deployment and rollouts
- Automatic scaling (horizontal and vertical)
- Self-healing (restarts failed containers)
- Service discovery and load balancing
- Storage orchestration
- Secret and configuration management
- Batch execution support

### Key Benefits

1. **Portability**: Run anywhere (cloud, on-prem, hybrid)
2. **Scalability**: Scale up/down automatically
3. **High Availability**: Self-healing and redundancy
4. **Resource Efficiency**: Optimal resource utilization
5. **Declarative Configuration**: Describe desired state
6. **Extensibility**: Rich ecosystem and APIs

## 📚 Core Architecture

### Cluster Components

```
┌─────────────────────────────────────────────────────────┐
│                    Control Plane                          │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐│
│  │ API      │  │ etcd     │  │Scheduler │  │Controller││
│  │ Server   │  │          │  │          │  │ Manager  ││
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘│
│       │             │             │             │         │
└───────┼─────────────┼─────────────┼─────────────┼─────────┘
        │             │             │             │
        └─────────────┴─────────────┴─────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────────┐
│                    Worker Nodes                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐            │
│  │ kubelet  │  │kube-proxy│  │ Container│            │
│  │          │  │          │  │ Runtime   │            │
│  └──────────┘  └──────────┘  └──────────┘            │
│       │             │             │                     │
│       └─────────────┴─────────────┘                     │
│                    │                                    │
│                    ▼                                    │
│              ┌──────────┐                               │
│              │   Pods   │                               │
│              └──────────┘                               │
└─────────────────────────────────────────────────────────┘
```

### Control Plane Components

#### 1. API Server (kube-apiserver)
- **Purpose**: Front-end for Kubernetes control plane
- **Functions**:
  - Exposes Kubernetes API
  - Validates and processes API requests
  - Updates etcd with cluster state
  - Handles authentication and authorization
- **Access**: REST API or `kubectl` CLI

#### 2. etcd
- **Purpose**: Distributed key-value store
- **Functions**:
  - Stores cluster configuration
  - Stores desired and actual state
  - Backend for service discovery
- **Characteristics**: Highly available, consistent

#### 3. Scheduler (kube-scheduler)
- **Purpose**: Assigns pods to nodes
- **Functions**:
  - Watches for unscheduled pods
  - Evaluates node resources
  - Applies scheduling policies
  - Binds pods to nodes

#### 4. Controller Manager (kube-controller-manager)
- **Purpose**: Runs controller processes
- **Controllers**:
  - Node Controller
  - Replication Controller
  - Endpoints Controller
  - Service Account Controller

#### 5. Cloud Controller Manager (optional)
- **Purpose**: Links cluster to cloud provider
- **Functions**:
  - Node lifecycle management
  - Route management
  - Service management
  - Volume management

### Worker Node Components

#### 1. kubelet
- **Purpose**: Agent running on each node
- **Functions**:
  - Registers node with API server
  - Monitors pods assigned to node
  - Reports node and pod status
  - Executes pod lifecycle actions

#### 2. kube-proxy
- **Purpose**: Network proxy on each node
- **Functions**:
  - Maintains network rules
  - Enables service abstraction
  - Load balancing for services
  - Implements service types

#### 3. Container Runtime
- **Purpose**: Runs containers
- **Supported Runtimes**:
  - containerd (default)
  - CRI-O
  - Docker (via containerd)
  - Mirantis Container Runtime

## 📦 Core Concepts

### Pods

**Pod** is the smallest deployable unit in Kubernetes. A pod contains one or more containers that share:
- Storage volumes
- Network namespace
- IP address
- Lifecycle

#### Single Container Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
    tier: frontend
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
      protocol: TCP
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
    env:
    - name: ENV_VAR
      value: "production"
    volumeMounts:
    - name: config
      mountPath: /etc/nginx/conf.d
  volumes:
  - name: config
    configMap:
      name: nginx-config
```

#### Multi-Container Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-app
spec:
  containers:
  - name: web
    image: nginx:1.21
    ports:
    - containerPort: 80
  - name: sidecar
    image: busybox
    command: ['sh', '-c', 'while true; do echo "Sidecar running"; sleep 30; done']
```

**Use Cases for Multi-Container Pods**:
- Sidecar pattern (logging, monitoring)
- Adapter pattern (data transformation)
- Ambassador pattern (proxy, service mesh)

### Deployments

**Deployment** manages a set of identical pods. It provides:
- Declarative updates
- Rolling updates and rollbacks
- Scaling
- Self-healing

#### Basic Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
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
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
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

#### Deployment Strategies

**Rolling Update (Default)**:
```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
```

**Recreate**:
```yaml
spec:
  strategy:
    type: Recreate
```

### Services

**Service** provides stable network access to pods. It abstracts pod IP addresses which change.

#### Service Types

**1. ClusterIP (Default)**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
```

**2. NodePort**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
    protocol: TCP
```

**3. LoadBalancer**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-lb
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
```

**4. ExternalName**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  type: ExternalName
  externalName: database.example.com
```

### ConfigMaps

**ConfigMap** stores non-confidential configuration data.

#### Creating ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: "postgresql://db:5432/mydb"
  log_level: "info"
  config.yaml: |
    server:
      port: 8080
      host: 0.0.0.0
```

#### Using ConfigMap

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp:latest
    envFrom:
    - configMapRef:
        name: app-config
    # OR
    env:
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database_url
    volumeMounts:
    - name: config
      mountPath: /etc/config
  volumes:
  - name: config
    configMap:
      name: app-config
```

### Secrets

**Secret** stores sensitive data (passwords, tokens, keys).

#### Creating Secret

```bash
# From literal
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=secret123

# From file
kubectl create secret generic db-secret \
  --from-file=username=./username.txt \
  --from-file=password=./password.txt
```

#### Secret Manifest

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=  # base64 encoded
  password: c2VjcmV0MTIz
```

#### Using Secret

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp:latest
    env:
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: username
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: db-secret
```

### Namespaces

**Namespace** provides logical separation and resource isolation.

#### Default Namespaces

- `default`: Default namespace
- `kube-system`: System components
- `kube-public`: Publicly accessible data
- `kube-node-lease`: Node heartbeat

#### Creating Namespace

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    name: production
    environment: prod
```

```bash
kubectl create namespace production
kubectl create namespace development
```

#### Using Namespaces

```bash
# List resources in namespace
kubectl get pods -n production

# Apply to namespace
kubectl apply -f deployment.yaml -n production

# Set default namespace
kubectl config set-context --current --namespace=production
```

## 🔧 Essential kubectl Commands

### Resource Management

```bash
# Get resources
kubectl get pods
kubectl get pods -A                    # All namespaces
kubectl get pods -n production         # Specific namespace
kubectl get pods -o wide              # More details
kubectl get pods -o yaml              # YAML output
kubectl get pods -o json              # JSON output
kubectl get pods --watch              # Watch changes
kubectl get pods -l app=nginx         # Filter by label

# Describe resource
kubectl describe pod nginx-pod
kubectl describe deployment nginx-deployment
kubectl describe service nginx-service

# Create resources
kubectl create deployment nginx --image=nginx:1.21
kubectl create namespace production
kubectl create secret generic my-secret --from-literal=key=value

# Apply manifests
kubectl apply -f deployment.yaml
kubectl apply -f .                     # All files in directory
kubectl apply -f https://example.com/manifest.yaml

# Delete resources
kubectl delete pod nginx-pod
kubectl delete deployment nginx-deployment
kubectl delete -f deployment.yaml
kubectl delete pods --all             # All pods
kubectl delete pods -l app=nginx      # By label
```

### Debugging & Inspection

```bash
# View logs
kubectl logs nginx-pod
kubectl logs nginx-pod -c sidecar     # Multi-container pod
kubectl logs -f nginx-pod             # Follow logs
kubectl logs --previous nginx-pod     # Previous container instance
kubectl logs deployment/nginx-deployment  # All pods in deployment

# Execute commands
kubectl exec -it nginx-pod -- bash
kubectl exec nginx-pod -- ls /var/log
kubectl exec nginx-pod -c sidecar -- sh

# Port forwarding
kubectl port-forward pod/nginx-pod 8080:80
kubectl port-forward service/nginx-service 8080:80
kubectl port-forward deployment/nginx-deployment 8080:80

# Copy files
kubectl cp nginx-pod:/etc/nginx/nginx.conf ./nginx.conf
kubectl cp ./file.txt nginx-pod:/tmp/file.txt
```

### Scaling & Updates

```bash
# Scale deployment
kubectl scale deployment nginx-deployment --replicas=5
kubectl scale --replicas=3 deployment/nginx-deployment

# Rolling update
kubectl set image deployment/nginx-deployment nginx=nginx:1.22
kubectl rollout status deployment/nginx-deployment
kubectl rollout history deployment/nginx-deployment
kubectl rollout undo deployment/nginx-deployment
kubectl rollout undo deployment/nginx-deployment --to-revision=2

# Update resources
kubectl edit deployment nginx-deployment
kubectl patch deployment nginx-deployment -p '{"spec":{"replicas":5}}'
```

### Resource Information

```bash
# Cluster information
kubectl cluster-info
kubectl get nodes
kubectl get nodes -o wide
kubectl describe node node-name
kubectl top nodes                      # Resource usage
kubectl top pods                       # Pod resource usage

# API resources
kubectl api-resources
kubectl api-versions
kubectl explain pod                    # Documentation
kubectl explain pod.spec.containers

# Events
kubectl get events
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl get events -n production
```

### Context & Configuration

```bash
# Context management
kubectl config get-contexts
kubectl config current-context
kubectl config use-context my-context
kubectl config set-context my-context --namespace=production

# View configuration
kubectl config view
kubectl config view --minify
kubectl config get-clusters
kubectl config get-users
```

## 🎯 Common Patterns & Use Cases

### Health Checks

#### Liveness Probe
Detects if container is running. If fails, container is restarted.

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3
```

#### Readiness Probe
Detects if container is ready to serve traffic. If fails, pod is removed from service endpoints.

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

#### Startup Probe
Detects if container has started. Useful for slow-starting containers.

```yaml
startupProbe:
  httpGet:
    path: /startup
    port: 8080
  failureThreshold: 30
  periodSeconds: 10
```

### Resource Management

#### Requests and Limits

```yaml
resources:
  requests:
    memory: "128Mi"
    cpu: "250m"        # 0.25 CPU cores
  limits:
    memory: "256Mi"
    cpu: "500m"        # 0.5 CPU cores
```

**CPU Units**:
- `1` = 1 CPU core
- `1000m` = 1 CPU core
- `500m` = 0.5 CPU core

**Memory Units**:
- `128Mi` = 128 Mebibytes
- `1Gi` = 1 Gibibyte
- `512M` = 512 Megabytes

### Init Containers

Init containers run before main containers and must complete successfully.

```yaml
spec:
  initContainers:
  - name: init-db
    image: busybox
    command: ['sh', '-c', 'until nslookup database; do echo waiting; sleep 2; done']
  containers:
  - name: app
    image: myapp:latest
```

### Lifecycle Hooks

```yaml
containers:
- name: app
  image: myapp:latest
  lifecycle:
    postStart:
      exec:
        command: ['/bin/sh', '-c', 'echo "Container started"']
    preStop:
      exec:
        command: ['/bin/sh', '-c', 'sleep 15']
```

## 🚀 Getting Started

### Installation Options

#### Minikube (Local Development)

```bash
# Install minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Start cluster
minikube start
minikube start --driver=docker
minikube start --nodes=3  # Multi-node

# Verify
kubectl get nodes
minikube status
```

#### kind (Kubernetes in Docker)

```bash
# Install kind
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

# Create cluster
kind create cluster
kind create cluster --name my-cluster

# Verify
kubectl cluster-info --context kind-my-cluster
```

#### k3d (Lightweight k3s)

```bash
# Install k3d
curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash

# Create cluster
k3d cluster create my-cluster
k3d cluster create my-cluster --servers 3 --agents 2

# Verify
kubectl get nodes
```

### First Deployment

```bash
# 1. Create deployment
kubectl create deployment nginx --image=nginx:1.21

# 2. Scale deployment
kubectl scale deployment nginx --replicas=3

# 3. Expose service
kubectl expose deployment nginx --port=80 --type=NodePort

# 4. Get service URL
minikube service nginx --url

# 5. View resources
kubectl get all
```

### Complete Example

```bash
# Create namespace
kubectl create namespace demo

# Apply deployment
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  namespace: demo
spec:
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
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
---
apiVersion: v1
kind: Service
metadata:
  name: web-service
  namespace: demo
spec:
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
  type: LoadBalancer
EOF

# Verify
kubectl get all -n demo
```

## 📊 Monitoring & Observability

### Resource Usage

```bash
# Install metrics server (if not installed)
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# View resource usage
kubectl top nodes
kubectl top pods
kubectl top pods --containers
kubectl top pods -n production
```

### Logs Aggregation

```bash
# View logs from all pods
kubectl logs -l app=nginx --all-containers=true

# Follow logs
kubectl logs -f deployment/nginx-deployment

# Previous instance logs
kubectl logs --previous pod/nginx-pod
```

## 🔍 Troubleshooting

### Pod Issues

```bash
# Pod not starting
kubectl describe pod pod-name
kubectl logs pod-name
kubectl get events --sort-by=.metadata.creationTimestamp

# Pod in CrashLoopBackOff
kubectl logs pod-name --previous
kubectl describe pod pod-name | grep -A 10 Events

# Pod not ready
kubectl get pods -o wide
kubectl describe pod pod-name
kubectl exec pod-name -- cat /etc/resolv.conf
```

### Service Issues

```bash
# Service not accessible
kubectl get svc
kubectl describe svc service-name
kubectl get endpoints service-name
kubectl get pods -l app=nginx

# DNS issues
kubectl exec -it pod-name -- nslookup service-name
kubectl exec -it pod-name -- ping service-name
```

### Network Issues

```bash
# Check network policies
kubectl get networkpolicies
kubectl describe networkpolicy policy-name

# Test connectivity
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup service-name
kubectl run -it --rm debug --image=busybox --restart=Never -- wget -O- http://service-name:80
```

## ✅ Best Practices

1. **Use Deployments, not Pods**: Deployments provide management capabilities
2. **Set Resource Limits**: Prevent resource exhaustion
3. **Use Health Checks**: Enable self-healing
4. **Label Everything**: Organize and select resources
5. **Use Namespaces**: Isolate environments
6. **Store Secrets Properly**: Use Kubernetes secrets or external secret management
7. **Version Your Manifests**: Use Git for version control
8. **Use ConfigMaps for Config**: Separate config from code
9. **Implement Resource Quotas**: Prevent resource abuse
10. **Monitor Everything**: Use metrics and logging

## 🎓 Learning Path

1. **Week 1**: Understand pods, deployments, services
2. **Week 2**: Learn ConfigMaps, Secrets, Namespaces
3. **Week 3**: Master kubectl commands
4. **Week 4**: Practice troubleshooting
5. **Week 5+**: Advanced topics (networking, storage, security)

## 📚 Additional Resources

- **Official Docs**: https://kubernetes.io/docs/
- **kubectl Cheat Sheet**: https://kubernetes.io/docs/reference/kubectl/cheatsheet/
- **Interactive Tutorial**: https://kubernetes.io/docs/tutorials/
- **Playground**: https://www.katacoda.com/courses/kubernetes

---

**Next Steps**: 
- Learn [Kubernetes Operations](./kubernetes-operations.md)
- Understand [Kubernetes Networking](./kubernetes-networking.md)
- Master [Kubernetes Storage](./kubernetes-storage.md)
- Explore [Service Mesh](./service-mesh.md)

**Remember**: Kubernetes is powerful but complex. Start with basics, practice regularly, and gradually explore advanced features. Hands-on experience is essential for mastery.
