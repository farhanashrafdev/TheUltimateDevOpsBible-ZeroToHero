# Container Fundamentals

## 🎯 Introduction

Containers package applications with all dependencies, ensuring consistency across development, testing, and production environments. This guide covers container concepts beyond Docker.

## 📚 Container Concepts

### Container Architecture

```
┌─────────────────────────────┐
│      Application            │
│      Runtime                │
│      Libraries              │
│      System Tools           │
├─────────────────────────────┤
│   Container Runtime         │
│   (containerd, CRI-O)       │
├─────────────────────────────┤
│   Operating System          │
│   (Linux Kernel)            │
└─────────────────────────────┘
```

### Container vs Virtual Machine

**Containers**:
- Share host OS kernel
- Lightweight
- Fast startup
- Less isolation

**Virtual Machines**:
- Full OS per VM
- Heavier
- Slower startup
- More isolation

## 🐳 Container Runtimes

### containerd
- Industry standard
- Used by Docker, Kubernetes
- OCI-compliant

### CRI-O
- Kubernetes-native
- Lightweight
- OCI-compliant

### Podman
- Rootless containers
- Docker-compatible
- No daemon

## 📦 Container Images

### Image Layers

```
┌─────────────────┐
│   Application   │  ← Top layer (writable)
├─────────────────┤
│   Dependencies  │  ← Layer 3
├─────────────────┤
│   Base System   │  ← Layer 2
├─────────────────┤
│   Base Image    │  ← Layer 1 (read-only)
└─────────────────┘
```

### Image Optimization

1. **Multi-stage builds**: Reduce final image size
2. **Layer caching**: Optimize build speed
3. **Minimal base images**: Alpine Linux
4. **Remove unnecessary files**: Clean up in same layer
5. **Use .dockerignore**: Exclude files

## 🔧 Container Best Practices

### Security
- Run as non-root user
- Scan images for vulnerabilities
- Use minimal base images
- Keep images updated
- Don't store secrets in images

### Performance
- Optimize image size
- Use layer caching
- Minimize layers
- Use specific tags

### Maintainability
- Clear Dockerfiles
- Document images
- Version tags
- Regular updates

## 🎯 Common Patterns

### Development Container

```dockerfile
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["npm", "run", "dev"]
```

### Production Container

```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
USER node
CMD ["node", "server.js"]
```

## ✅ Key Takeaways

- Containers provide consistency and portability
- Use multi-stage builds for optimization
- Follow security best practices
- Optimize for size and performance

---

**Next**: Learn CI/CD for automated container builds and deployments.

