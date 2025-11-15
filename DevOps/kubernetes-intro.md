# Kubernetes Introduction

## 🎯 What is Kubernetes?

Kubernetes (K8s) is an open-source container orchestration platform for automating deployment, scaling, and management of containerized applications.

## 📚 Core Concepts

### Cluster Architecture
```
┌─────────────────────────────────┐
│      Control Plane              │
│  (API Server, etcd, Scheduler)  │
└─────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────┐
│      Worker Nodes               │
│  (kubelet, kube-proxy, pods)    │
└─────────────────────────────────┘
```

### Key Components
- **Pods**: Smallest deployable units
- **Deployments**: Manage pod replicas
- **Services**: Network access to pods
- **Namespaces**: Virtual clusters
- **ConfigMaps**: Configuration data
- **Secrets**: Sensitive data

## 📝 Basic Resources

### Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
```

### Deployment
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

### Service
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
    targetPort: 80
  type: ClusterIP
```

## 🔧 Basic Commands

```bash
# Get resources
kubectl get pods
kubectl get deployments
kubectl get services

# Describe resource
kubectl describe pod nginx-pod

# Apply manifest
kubectl apply -f deployment.yaml

# Delete resource
kubectl delete pod nginx-pod

# Logs
kubectl logs nginx-pod

# Execute command
kubectl exec -it nginx-pod -- bash
```

## ✅ Key Takeaways

- Kubernetes orchestrates containers
- Pods are basic units
- Deployments manage replicas
- Services provide networking
- Use namespaces for organization

---

**Next**: Learn Kubernetes operations and advanced topics.

