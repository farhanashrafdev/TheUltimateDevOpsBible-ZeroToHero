# Container Security: Complete Guide

## 🎯 Introduction

Container security requires a multi-layered approach covering build time, image security, runtime protection, and supply chain security. This guide covers all aspects of securing containers.

### Container Security Layers

```
┌─────────────────────────────────────────┐
│      Container Security Layers            │
│                                           │
│  ┌─────────────────────────────────┐    │
│  │   Supply Chain Security         │    │
│  │   (SBOM, Signing, Provenance)   │    │
│  └─────────────────────────────────┘    │
│  ┌─────────────────────────────────┐    │
│  │   Runtime Security               │    │
│  │   (Falco, Monitoring, Policies) │    │
│  └─────────────────────────────────┘    │
│  ┌─────────────────────────────────┐    │
│  │   Image Security                 │    │
│  │   (Scanning, Signing, Minimal)   │    │
│  └─────────────────────────────────┘    │
│  ┌─────────────────────────────────┐    │
│  │   Build Security                 │    │
│  │   (Dockerfile, Non-root, Minimal)│    │
│  └─────────────────────────────────┘    │
└─────────────────────────────────────────┘
```

## 🏗️ Build-Time Security

### Secure Dockerfiles

#### Best Practices

```dockerfile
# Use specific tags, not latest
FROM node:20-alpine

# Create non-root user
RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser

# Set working directory
WORKDIR /app

# Copy package files first (layer caching)
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production && \
    npm cache clean --force

# Copy application code
COPY . .

# Remove unnecessary packages
RUN apk del .build-deps

# Switch to non-root user
USER appuser

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD node healthcheck.js

# Use exec form for CMD
CMD ["node", "server.js"]
```

#### Multi-Stage Builds

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
RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./
RUN npm ci --only=production
USER appuser
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

### Security Hardening

```dockerfile
# Remove unnecessary tools
RUN apk del curl wget

# Set read-only root filesystem (in Kubernetes)
# securityContext:
#   readOnlyRootFilesystem: true

# Drop capabilities
# securityContext:
#   capabilities:
#     drop:
#       - ALL
#     add:
#       - NET_BIND_SERVICE
```

## 🔍 Image Security

### Image Scanning

#### Trivy

**Comprehensive vulnerability scanner**

```bash
# Install Trivy
brew install trivy
# or
docker pull aquasec/trivy

# Scan image
trivy image nginx:latest

# Scan with severity filter
trivy image --severity HIGH,CRITICAL nginx:latest

# Scan and output JSON
trivy image -f json -o results.json nginx:latest

# Scan filesystem
trivy fs .

# Scan Kubernetes cluster
trivy k8s cluster --report summary
```

**CI/CD Integration**:

```yaml
# GitHub Actions
name: Trivy Scan
on:
  push:
    branches: [ main ]

jobs:
  trivy-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build image
        run: docker build -t myapp:latest .
      
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'myapp:latest'
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH'
      
      - name: Upload Trivy results to GitHub Security
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'
```

#### Snyk

**Container vulnerability scanning**

```bash
# Install Snyk
npm install -g snyk

# Authenticate
snyk auth

# Test container image
snyk container test nginx:latest

# Monitor image
snyk container monitor nginx:latest

# Test with policy
snyk container test nginx:latest --policy-path=.snyk
```

#### Clair

**Static analysis of container images**

```bash
# Run Clair
docker run -d --name clair \
  -p 6060-6061:6060-6061 \
  quay.io/projectclair/clair:latest

# Scan image
clair-scanner --clair=http://localhost:6060 \
  --ip=host.docker.internal nginx:latest
```

### Minimal Base Images

**Image Size Comparison**:
- **Alpine**: ~5MB
- **Distroless**: ~10-20MB
- **Ubuntu**: ~70MB
- **Debian**: ~120MB

**Use Alpine**:
```dockerfile
FROM alpine:latest
RUN apk add --no-cache nodejs npm
```

**Use Distroless**:
```dockerfile
FROM gcr.io/distroless/nodejs:20
COPY . .
CMD ["server.js"]
```

### Image Signing

**Sign images for integrity**

```bash
# Install Cosign
go install github.com/sigstore/cosign/v2/cmd/cosign@latest

# Generate key pair
cosign generate-key-pair

# Sign image
cosign sign --key cosign.key user/image:tag

# Verify signature
cosign verify --key cosign.pub user/image:tag

# Keyless signing (CI/CD)
cosign sign user/image:tag
cosign verify user/image:tag
```

## 🛡️ Runtime Security

### Falco

**Runtime threat detection**

#### Installation

```bash
# Install Falco
curl -s https://falco.org/repo/falcosecurity-3672BA8F.asc | sudo apt-key add -
echo "deb https://download.falco.org/packages/deb stable main" | sudo tee -a /etc/apt/sources.list.d/falcosecurity.list
sudo apt-get update
sudo apt-get install -y falco

# Start Falco
sudo systemctl start falco
sudo systemctl enable falco
```

#### Kubernetes Installation

```bash
# Install Falco in Kubernetes
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm repo update
helm install falco falcosecurity/falco
```

#### Custom Rules

```yaml
# falco_rules.yaml
- rule: Write below binary dir
  desc: Detect writes to binary directories
  condition: >
    bin_dir and evt.dir = < and open_write
  output: >
    File below a known binary directory opened for writing
    (user=%user.name command=%proc.cmdline file=%fd.name)
  priority: ERROR
  tags: [filesystem, mitre_persistence]

- rule: Unexpected network activity
  desc: Detect unexpected network connections
  condition: >
    evt.type = connect and
    container.id != host and
    not fd.sip in (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16)
  output: >
    Unexpected network connection
    (user=%user.name command=%proc.cmdline connection=%fd.name)
  priority: WARNING
  tags: [network]
```

### Runtime Policies

#### Kubernetes Security Context

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 1000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: myapp:latest
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
          - ALL
        add:
          - NET_BIND_SERVICE
      runAsNonRoot: true
      runAsUser: 1000
```

#### Pod Security Standards

```yaml
# Enforce Pod Security Standards
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

### Network Policies

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: app-network-policy
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
  - to: []
    ports:
    - protocol: TCP
      port: 443
    - protocol: TCP
      port: 80
```

## 🔗 Supply Chain Security

### SBOM Generation

**Software Bill of Materials**

```bash
# Install Syft
curl -sSfL https://raw.githubusercontent.com/anchore/syft/main/install.sh | sh -s -- -b /usr/local/bin

# Generate SBOM
syft packages docker:nginx:latest -o spdx-json > sbom.json

# Generate for directory
syft packages dir:. -o cyclonedx-json > sbom.json
```

### Image Provenance

**SLSA Provenance**

```bash
# Generate provenance with SLSA
slsa-verifier verify-image \
  --source-uri github.com/user/repo \
  --source-tag v1.0.0 \
  user/image:tag
```

## 📊 Security Monitoring

### Container Security Metrics

- **Vulnerability Count**: By severity
- **Compliance Score**: Policy compliance
- **Runtime Alerts**: Falco alerts
- **Image Age**: Outdated images
- **Scan Coverage**: Percentage scanned

### Security Dashboards

```yaml
# Grafana Dashboard
- Vulnerability trends
- Runtime alerts
- Compliance status
- Image security scores
- Remediation progress
```

## ✅ Best Practices

### 1. Scan All Images

```yaml
# Scan in CI/CD
# Scan before deployment
# Scan in registries
# Regular scanning
```

### 2. Use Minimal Base Images

```yaml
# Alpine Linux
# Distroless images
# Remove unnecessary packages
# Multi-stage builds
```

### 3. Run as Non-Root

```yaml
# Create non-root users
# Set security contexts
# Drop capabilities
# Read-only filesystems
```

### 4. Implement Runtime Protection

```yaml
# Falco rules
# Network policies
# Pod security standards
# Monitoring
```

### 5. Regular Updates

```yaml
# Update base images
# Patch vulnerabilities
# Update dependencies
# Security patches
```

### 6. Sign Images

```yaml
# Sign all images
# Verify signatures
# Keyless signing in CI/CD
# Signature policies
```

## ✅ Mastery Checklist

- [ ] Secure Dockerfiles
- [ ] Scan container images
- [ ] Use minimal base images
- [ ] Sign container images
- [ ] Implement runtime protection
- [ ] Set up Falco
- [ ] Configure network policies
- [ ] Generate SBOMs
- [ ] Monitor security
- [ ] Regular updates

---

**Next Steps**:
- Learn [Kubernetes Runtime Security](./k8s-runtime-security.md)
- Explore [Supply Chain Security](./supply-chain-security.md)
- Master [Image Signing](./image-signing.md)

**Remember**: Container security requires defense in depth. Secure builds, scan images, protect runtime, and monitor continuously. Start with basics, implement best practices, and continuously improve your security posture.
