# Container Fundamentals: Complete Guide

## 🎯 Introduction

Containers have revolutionized software deployment by providing a consistent, portable way to package applications with all their dependencies. This guide covers container concepts, runtimes, and best practices beyond Docker.

### What are Containers?

Containers are lightweight, portable units that package an application and its dependencies, ensuring it runs consistently across different environments.

**Key Characteristics**:
- **Isolation**: Process and filesystem isolation
- **Portability**: Run anywhere (dev, test, prod)
- **Efficiency**: Share host OS kernel
- **Consistency**: Same behavior everywhere
- **Scalability**: Easy to scale horizontally

## 📚 Container Architecture

### Container vs Virtual Machine

```
Virtual Machine:
┌─────────────────────────────────┐
│        Application              │
│        Bins/Libs                │
│        Guest OS (Full)          │
│    ┌──────────────────────┐    │
│    │   Hypervisor         │    │
│    └──────────────────────┘    │
│        Host OS                  │
│        Hardware                 │
└─────────────────────────────────┘

Container:
┌─────────────────────────────────┐
│        Application              │
│        Bins/Libs                │
│    ┌──────────────────────┐    │
│    │   Container Runtime   │    │
│    │   (containerd/CRI-O)  │    │
│    └──────────────────────┘    │
│        Host OS (Shared)         │
│        Hardware                 │
└─────────────────────────────────┘
```

**Comparison**:

| Aspect | Containers | Virtual Machines |
|--------|-----------|------------------|
| **OS** | Share host kernel | Full OS per VM |
| **Size** | MBs | GBs |
| **Startup** | Seconds | Minutes |
| **Isolation** | Process level | Hardware level |
| **Resource Usage** | Lower | Higher |
| **Portability** | High | Medium |

### Container Components

```
Container Image
├── Base Layer (OS files)
├── Dependency Layer (libraries)
├── Application Layer (code)
└── Configuration Layer (configs)
```

## 🐳 Container Runtimes

### containerd

**Industry-standard container runtime**

```bash
# Install containerd
# Ubuntu/Debian
sudo apt-get update
sudo apt-get install -y containerd

# Start containerd
sudo systemctl start containerd
sudo systemctl enable containerd

# Verify
sudo ctr version
```

**Features**:
- OCI-compliant
- Used by Docker and Kubernetes
- Supports multiple image formats
- Plugin architecture

### CRI-O

**Kubernetes-native container runtime**

```bash
# Install CRI-O
# Ubuntu
OS=xUbuntu_22.04
VERSION=1.28

echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /" | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
echo "deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/ /" | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.list

curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/Release.key | sudo apt-key add -
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | sudo apt-key add -

sudo apt-get update
sudo apt-get install -y cri-o cri-o-runc

# Start CRI-O
sudo systemctl start crio
sudo systemctl enable crio
```

**Features**:
- Built for Kubernetes
- Lightweight
- OCI-compliant
- No Docker dependency

### Podman

**Rootless container engine**

```bash
# Install Podman
# Ubuntu
sudo apt-get update
sudo apt-get install -y podman

# Run container (no root needed)
podman run -d nginx:latest

# List containers
podman ps

# Docker-compatible commands
podman build -t myapp .
podman push myapp:latest
```

**Features**:
- Rootless by default
- Docker-compatible CLI
- No daemon required
- Better security

## 📦 Container Images

### Image Layers

```
┌─────────────────────────────┐
│   Application Code          │  ← Layer 4 (writable)
├─────────────────────────────┤
│   Application Dependencies  │  ← Layer 3
├─────────────────────────────┤
│   System Libraries          │  ← Layer 2
├─────────────────────────────┤
│   Base OS Image             │  ← Layer 1 (read-only)
└─────────────────────────────┘
```

**Layer Benefits**:
- **Caching**: Unchanged layers are cached
- **Sharing**: Multiple images share base layers
- **Efficiency**: Only changed layers are rebuilt

### Image Optimization

#### 1. Multi-Stage Builds

```dockerfile
# Build stage
FROM node:20 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./
RUN npm ci --only=production
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

#### 2. Minimal Base Images

```dockerfile
# Good - Alpine Linux (5MB)
FROM alpine:latest

# Better - Distroless (even smaller)
FROM gcr.io/distroless/nodejs:20

# Avoid - Full OS (hundreds of MB)
FROM ubuntu:latest
```

#### 3. Layer Optimization

```dockerfile
# Bad - Multiple layers
RUN apt-get update
RUN apt-get install -y package1
RUN apt-get install -y package2
RUN apt-get clean

# Good - Single layer
RUN apt-get update && \
    apt-get install -y package1 package2 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

#### 4. .dockerignore

```dockerignore
# .dockerignore
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
.nyc_output
coverage
.vscode
.idea
*.md
```

## 🔒 Container Security

### Security Best Practices

#### 1. Run as Non-Root

```dockerfile
# Create non-root user
RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser

USER appuser

# Or use distroless (no shell, no root)
FROM gcr.io/distroless/nodejs:20
```

#### 2. Scan Images

```bash
# Trivy scan
trivy image myapp:latest

# Snyk scan
snyk container test myapp:latest

# Docker Scout
docker scout cves myapp:latest
```

#### 3. Minimal Base Images

```dockerfile
# Use minimal images
FROM alpine:latest
# or
FROM distroless/nodejs:20
```

#### 4. Don't Store Secrets

```dockerfile
# Bad - Secrets in image
ENV API_KEY=secret123

# Good - Use secrets management
# Pass at runtime or use secret mounts
```

#### 5. Read-Only Root Filesystem

```yaml
# Kubernetes
securityContext:
  readOnlyRootFilesystem: true
  runAsNonRoot: true
  runAsUser: 1000
```

### Container Isolation

#### Namespaces

- **PID**: Process isolation
- **Network**: Network isolation
- **Mount**: Filesystem isolation
- **UTS**: Hostname isolation
- **IPC**: Inter-process communication
- **User**: User ID isolation

#### cgroups

- **CPU**: CPU limits
- **Memory**: Memory limits
- **I/O**: Disk I/O limits
- **Devices**: Device access control

## 🚀 Container Patterns

### Sidecar Pattern

```yaml
# Kubernetes Pod with Sidecar
apiVersion: v1
kind: Pod
metadata:
  name: app-with-sidecar
spec:
  containers:
  - name: app
    image: myapp:latest
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log
  - name: log-collector
    image: fluentd:latest
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log
  volumes:
  - name: shared-logs
    emptyDir: {}
```

### Init Container Pattern

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-init
spec:
  initContainers:
  - name: init-db
    image: busybox
    command: ['sh', '-c', 'until nslookup database; do echo waiting; sleep 2; done']
  containers:
  - name: app
    image: myapp:latest
```

### Ambassador Pattern

```yaml
# Proxy container
containers:
- name: app
  image: myapp:latest
- name: proxy
  image: envoy:latest
  # Routes traffic
```

## 📊 Container Orchestration

### Why Orchestration?

- **Scaling**: Scale containers up/down
- **High Availability**: Restart failed containers
- **Load Balancing**: Distribute traffic
- **Service Discovery**: Find containers
- **Rolling Updates**: Zero-downtime deployments
- **Resource Management**: Optimize resource usage

### Orchestration Tools

1. **Kubernetes**: Industry standard
2. **Docker Swarm**: Docker-native
3. **Nomad**: HashiCorp's orchestrator
4. **ECS**: AWS container service
5. **Azure Container Instances**: Azure service

## 🔧 Container Best Practices

### Development

```dockerfile
# Development Dockerfile
FROM node:20
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "run", "dev"]
```

### Production

```dockerfile
# Production Dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine
WORKDIR /app
RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./
RUN npm ci --only=production
USER appuser
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s \
  CMD node healthcheck.js
CMD ["node", "dist/index.js"]
```

### Image Tagging

```bash
# Semantic versioning
docker build -t myapp:1.0.0 .
docker build -t myapp:1.0 .
docker build -t myapp:1 .
docker build -t myapp:latest .

# Git commit SHA
docker build -t myapp:$(git rev-parse --short HEAD) .

# Build number
docker build -t myapp:${BUILD_NUMBER} .
```

## 📝 Container Registries

### Public Registries

- **Docker Hub**: docker.io
- **GitHub Container Registry**: ghcr.io
- **Google Container Registry**: gcr.io
- **Amazon ECR**: Public Gallery

### Private Registries

```bash
# Docker Hub
docker login
docker push username/myapp:latest

# GitHub Container Registry
docker login ghcr.io -u USERNAME -p TOKEN
docker push ghcr.io/username/myapp:latest

# AWS ECR
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com
docker push 123456789012.dkr.ecr.us-east-1.amazonaws.com/myapp:latest
```

## ✅ Mastery Checklist

- [ ] Understand container concepts
- [ ] Use multiple container runtimes
- [ ] Optimize container images
- [ ] Implement security best practices
- [ ] Use multi-stage builds
- [ ] Understand container patterns
- [ ] Manage container registries
- [ ] Monitor container performance
- [ ] Troubleshoot container issues
- [ ] Scale containers effectively

---

**Next Steps**:
- Learn [Docker](./docker.md) in detail
- Master [Kubernetes](./kubernetes-intro.md) for orchestration
- Explore [Container Security](./DevSecOps/container-security.md)

**Remember**: Containers are the foundation of modern DevOps. Master image optimization, security practices, and orchestration. Start simple and gradually add complexity.
