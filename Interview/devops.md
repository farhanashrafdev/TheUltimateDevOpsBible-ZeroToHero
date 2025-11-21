# DevOps Interview Questions: Complete Guide

## 🎯 Introduction

This comprehensive guide covers DevOps interview questions from fundamentals to advanced scenarios, helping you prepare for technical interviews.

## 📚 Fundamentals

### What is DevOps?

**Answer**: DevOps is a cultural and technical movement that combines software development (Dev) and IT operations (Ops) to shorten the development lifecycle and provide continuous delivery with high quality. It emphasizes:

- **Culture**: Collaboration between teams
- **Automation**: Automate repetitive tasks
- **Measurement**: Metrics and monitoring
- **Sharing**: Knowledge sharing and collaboration

**Key Principles**:
1. Continuous Integration (CI)
2. Continuous Delivery/Deployment (CD)
3. Infrastructure as Code (IaC)
4. Monitoring and Logging
5. Microservices architecture
6. Containerization

### Explain CI/CD

**Answer**: 
- **CI (Continuous Integration)**: Automates code integration and testing. Developers frequently merge code changes into a central repository, where automated builds and tests run.

- **CD (Continuous Delivery)**: Code is always in a deployable state. After CI, code is automatically deployed to staging environments.

- **CD (Continuous Deployment)**: Extends Continuous Delivery by automatically deploying to production after passing tests.

**Benefits**:
- Faster feedback
- Early bug detection
- Reduced deployment risk
- Faster time to market

### What is Infrastructure as Code?

**Answer**: Infrastructure as Code (IaC) manages and provisions infrastructure through machine-readable definition files, rather than through manual configuration.

**Benefits**:
- Version control for infrastructure
- Reproducibility
- Consistency
- Faster provisioning
- Reduced errors

**Tools**: Terraform, CloudFormation, Pulumi, Ansible

## 🐳 Docker & Containers

### What is the difference between Docker image and container?

**Answer**:
- **Image**: A read-only template used to create containers. Contains application code, runtime, libraries, and dependencies.
- **Container**: A running instance of an image. Has a writable layer on top of the image.

**Analogy**: Image is like a class, container is like an object instance.

### Explain Docker layers

**Answer**: Docker images consist of layers. Each instruction in a Dockerfile creates a new layer:

```dockerfile
FROM node:18          # Layer 1: Base image
WORKDIR /app          # Layer 2: Working directory
COPY package.json .   # Layer 3: Copy files
RUN npm install       # Layer 4: Install dependencies
COPY . .              # Layer 5: Copy application
CMD ["node", "app.js"] # Layer 6: Command
```

**Benefits**:
- Layer caching (faster builds)
- Layer sharing (smaller images)
- Efficient storage

### How do you optimize Docker images?

**Answer**:
1. **Use multi-stage builds**: Reduce final image size
2. **Minimal base images**: Use Alpine or distroless
3. **Layer optimization**: Combine RUN commands
4. **Remove unnecessary files**: Clean up in same layer
5. **Use .dockerignore**: Exclude files from build
6. **Specific tags**: Avoid `latest` tag

**Example**:
```dockerfile
# Multi-stage build
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/index.js"]
```

### What is Docker Compose?

**Answer**: Docker Compose is a tool for defining and running multi-container Docker applications. Uses a YAML file to configure services.

**Use Cases**:
- Local development
- Testing environments
- Multi-container applications

**Example**:
```yaml
version: '3.8'
services:
  web:
    build: .
    ports:
      - "3000:3000"
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: password
```

## ☸️ Kubernetes

### What is a Pod?

**Answer**: A Pod is the smallest deployable unit in Kubernetes. It contains one or more containers that share:
- Network namespace (same IP)
- Storage volumes
- IPC namespace

**Characteristics**:
- Ephemeral (can be recreated)
- Single IP address
- Containers in pod can communicate via localhost

### Explain Kubernetes Services

**Answer**: Services provide stable network access to pods. Types:

1. **ClusterIP** (default): Internal only, accessible within cluster
2. **NodePort**: Exposes service on each node's IP at a static port
3. **LoadBalancer**: Exposes service externally using cloud load balancer
4. **ExternalName**: Maps service to external DNS name

**Why Services?**: Pods are ephemeral. Services provide stable endpoints.

### What are Deployments?

**Answer**: Deployments manage pod replicas and provide:
- Declarative updates
- Rolling updates
- Rollback capabilities
- Scaling

**Example**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
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

### Explain ConfigMaps and Secrets

**Answer**:
- **ConfigMaps**: Store non-confidential configuration data (key-value pairs, files)
- **Secrets**: Store sensitive data (passwords, tokens, keys)

**Usage**:
- Environment variables
- Volume mounts
- Command-line arguments

**Best Practices**:
- Never commit secrets
- Use external secret managers
- Rotate secrets regularly
- Encrypt secrets at rest

### What is Ingress?

**Answer**: Ingress provides HTTP/HTTPS routing to services. Features:
- URL-based routing
- SSL/TLS termination
- Load balancing
- Name-based virtual hosting

**Requires**: Ingress Controller (NGINX, Traefik, etc.)

### Explain Network Policies

**Answer**: Network Policies control pod-to-pod communication, enabling:
- Micro-segmentation
- Security isolation
- Traffic filtering

**Example**:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: app-policy
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
```

## 🔄 CI/CD

### How do you handle secrets in CI/CD?

**Answer**:
1. **Secret Managers**: Vault, AWS Secrets Manager, Azure Key Vault
2. **Never Commit**: Use .gitignore, pre-commit hooks
3. **Environment Variables**: Use CI/CD platform secrets
4. **Rotation**: Regular secret rotation
5. **Least Privilege**: Minimal access needed

**Example (GitHub Actions)**:
```yaml
- name: Use secret
  env:
    API_KEY: ${{ secrets.API_KEY }}
  run: echo "Using secret"
```

### Explain different deployment strategies

**Answer**:
1. **Rolling Update**: Gradual replacement of old pods
2. **Blue-Green**: Instant switch between environments
3. **Canary**: Gradual traffic shift to new version
4. **Recreate**: Terminate all old, then create new

**When to use**:
- Rolling: Standard deployments
- Blue-Green: Zero downtime, instant rollback
- Canary: Risk mitigation, gradual rollout
- Recreate: Stateful applications

### How do you implement CI/CD for Kubernetes?

**Answer**:
1. **GitOps**: Argo CD, Flux
2. **CI Pipeline**: Build, test, scan, build image
3. **CD Pipeline**: Deploy to Kubernetes
4. **GitOps Workflow**: Git as source of truth

**Example**:
```yaml
# CI: Build and push
- name: Build and push
  run: |
    docker build -t image:tag .
    docker push image:tag

# CD: Deploy with Argo CD
# GitOps: Argo CD watches Git repo
```

## ☁️ Infrastructure

### What is Terraform?

**Answer**: Terraform is an Infrastructure as Code tool that:
- Manages infrastructure across multiple clouds
- Uses declarative configuration (HCL)
- Maintains state
- Supports providers (AWS, Azure, GCP, etc.)

**Key Concepts**:
- Providers: Cloud/platform integrations
- Resources: Infrastructure components
- State: Current infrastructure state
- Modules: Reusable configurations

### Explain Terraform state

**Answer**: Terraform state:
- Tracks managed resources
- Maps configuration to real resources
- Enables updates and deletions
- Stores resource metadata

**State Management**:
- Local state (default)
- Remote state (S3, Terraform Cloud)
- State locking (DynamoDB)

### What is the difference between Terraform and Ansible?

**Answer**:
- **Terraform**: Infrastructure provisioning, declarative, stateful
- **Ansible**: Configuration management, imperative, agentless

**Use Together**:
- Terraform: Provision infrastructure
- Ansible: Configure provisioned infrastructure

## 📊 Monitoring & Observability

### Explain the three pillars of observability

**Answer**:
1. **Metrics**: Numerical measurements over time (CPU, memory, request rate)
2. **Logs**: Event records (application logs, access logs)
3. **Traces**: Request flow through services (distributed tracing)

**Tools**:
- Metrics: Prometheus
- Logs: ELK Stack, Loki
- Traces: Jaeger, Zipkin

### What is Prometheus?

**Answer**: Prometheus is a monitoring and alerting toolkit:
- Time-series database
- Pull-based metrics collection
- PromQL query language
- AlertManager integration

**Key Concepts**:
- Metrics: Exposed via /metrics endpoint
- Scraping: Prometheus pulls metrics
- Alerting: Based on PromQL queries

### How do you monitor Kubernetes?

**Answer**:
1. **Metrics Server**: Resource usage
2. **Prometheus**: Metrics collection
3. **Grafana**: Visualization
4. **Fluentd/Fluent Bit**: Log collection
5. **Jaeger**: Distributed tracing

## 🔒 Security

### How do you secure containers?

**Answer**:
1. **Image Scanning**: Trivy, Snyk, Clair
2. **Minimal Images**: Alpine, distroless
3. **Non-Root**: Run as non-root user
4. **Secrets Management**: External secret managers
5. **Network Policies**: Restrict communication
6. **Runtime Protection**: Falco, Aqua Security

### What is DevSecOps?

**Answer**: DevSecOps integrates security into DevOps:
- Shift-left security
- Security in CI/CD
- Automated security testing
- Continuous security monitoring

**Practices**:
- SAST (Static Analysis)
- SCA (Software Composition Analysis)
- DAST (Dynamic Analysis)
- Container scanning
- IaC scanning

## 🎯 Scenario Questions

### How would you troubleshoot a deployment failure?

**Answer**:
1. **Check Logs**: `kubectl logs pod-name`
2. **Describe Pod**: `kubectl describe pod pod-name`
3. **Check Events**: `kubectl get events`
4. **Verify Image**: Check if image exists and is accessible
5. **Check Resources**: CPU/memory limits
6. **Network**: Test connectivity
7. **Configuration**: Verify ConfigMaps/Secrets
8. **Dependencies**: Check dependent services

### Design a high-availability system

**Answer**:
1. **Multi-Region**: Deploy across regions
2. **Load Balancing**: Distribute traffic
3. **Auto-Scaling**: Handle traffic spikes
4. **Health Checks**: Monitor service health
5. **Database Replication**: Master-slave or multi-master
6. **Caching**: Redis/Memcached for performance
7. **CDN**: CloudFront for static content
8. **Disaster Recovery**: Backup and restore procedures
9. **Monitoring**: Comprehensive observability
10. **Incident Response**: Runbooks and procedures

### How do you handle a production incident?

**Answer**:
1. **Assess**: Determine severity and impact
2. **Communicate**: Notify stakeholders
3. **Investigate**: Check logs, metrics, recent changes
4. **Mitigate**: Apply quick fix if possible
5. **Resolve**: Implement permanent fix
6. **Verify**: Confirm resolution
7. **Post-Mortem**: Document and learn
8. **Prevent**: Implement measures to prevent recurrence

### Explain your CI/CD pipeline

**Answer** (Example):
1. **Source**: Git push triggers pipeline
2. **Build**: Compile code, run unit tests
3. **Test**: Integration tests, security scans
4. **Package**: Build Docker image
5. **Scan**: Image vulnerability scanning
6. **Deploy Staging**: Deploy to staging environment
7. **E2E Tests**: End-to-end testing
8. **Deploy Production**: Manual approval, deploy to prod
9. **Monitor**: Post-deployment monitoring

## ✅ Preparation Tips

1. **Practice Explaining**: Explain concepts clearly
2. **Prepare Examples**: Real-world examples from experience
3. **Review Troubleshooting**: Common issues and solutions
4. **System Design**: Practice designing systems
5. **Coding Challenges**: Practice scripting and automation
6. **Stay Updated**: Latest tools and practices
7. **Mock Interviews**: Practice with peers

## 🎯 Key Areas to Master

- [ ] Linux fundamentals
- [ ] Docker and containers
- [ ] Kubernetes
- [ ] CI/CD pipelines
- [ ] Infrastructure as Code
- [ ] Cloud platforms (AWS/GCP/Azure)
- [ ] Monitoring and observability
- [ ] Security practices
- [ ] Troubleshooting
- [ ] System design

---

**Remember**: DevOps interviews test both technical knowledge and problem-solving ability. Be prepared to explain concepts clearly, provide examples, and discuss real-world scenarios. Practice explaining complex topics simply.
