# DevOps Portfolio Projects: Complete Guide

## 🎯 Introduction

This guide provides comprehensive DevOps project ideas with detailed descriptions, architecture, implementation steps, and deliverables. These projects demonstrate real-world DevOps skills.

## 📚 Project Structure

Each project includes:
- **Description**: What you'll build
- **Architecture**: System design
- **Technologies**: Tools and services
- **Implementation Steps**: Step-by-step guide
- **Deliverables**: What to create
- **Challenges**: Advanced features
- **Portfolio Value**: Why it matters

## 🚀 Project 1: Multi-Tier Application with CI/CD

### Description

Deploy a full-stack application (frontend, backend, database) with complete CI/CD pipeline, containerization, and Kubernetes orchestration.

### Architecture

```
┌─────────────────────────────────────────┐
│         Application Architecture          │
│                                           │
│  ┌──────────┐      ┌──────────┐          │
│  │ Frontend │─────│ Backend  │          │
│  │ (React)  │     │(Node.js) │          │
│  └──────────┘      └────┬─────┘          │
│                         │                │
│                    ┌────▼─────┐          │
│                    │Database  │          │
│                    │(Postgres)│          │
│                    └──────────┘          │
│                                           │
│  CI/CD Pipeline:                          │
│  Git → GitHub Actions → Docker → K8s    │
└─────────────────────────────────────────┘
```

### Technologies

- **Frontend**: React, TypeScript
- **Backend**: Node.js, Express
- **Database**: PostgreSQL
- **Containerization**: Docker
- **Orchestration**: Kubernetes
- **CI/CD**: GitHub Actions
- **Monitoring**: Prometheus, Grafana

### Implementation Steps

#### Step 1: Application Setup

```bash
# Create project structure
mkdir fullstack-app
cd fullstack-app

# Frontend
npx create-react-app frontend --template typescript
cd frontend
npm install axios

# Backend
mkdir backend
cd backend
npm init -y
npm install express pg cors dotenv
npm install -D @types/express @types/node typescript ts-node

# Database
# Use PostgreSQL (local or cloud)
```

#### Step 2: Dockerize Applications

**Frontend Dockerfile**:
```dockerfile
# frontend/Dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**Backend Dockerfile**:
```dockerfile
# backend/Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
USER node
CMD ["node", "dist/server.js"]
```

#### Step 3: Kubernetes Manifests

```yaml
# k8s/frontend-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: myregistry/frontend:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
---
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 80
  type: LoadBalancer
```

```yaml
# k8s/backend-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: myregistry/backend:latest
        ports:
        - containerPort: 3000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: url
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
---
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
  - port: 3000
    targetPort: 3000
```

```yaml
# k8s/database.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:15
        env:
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: postgres-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

#### Step 4: CI/CD Pipeline

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  REGISTRY: docker.io
  IMAGE_NAME: ${{ secrets.DOCKER_USERNAME }}/fullstack-app

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: |
          cd frontend && npm ci
          cd ../backend && npm ci
      
      - name: Run tests
        run: |
          cd frontend && npm test -- --coverage --watchAll=false
          cd ../backend && npm test
      
      - name: Run linter
        run: |
          cd frontend && npm run lint
          cd ../backend && npm run lint

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      
      - name: Build and push frontend
        uses: docker/build-push-action@v4
        with:
          context: ./frontend
          push: true
          tags: ${{ env.IMAGE_NAME }}-frontend:${{ github.sha }},${{ env.IMAGE_NAME }}-frontend:latest
      
      - name: Build and push backend
        uses: docker/build-push-action@v4
        with:
          context: ./backend
          push: true
          tags: ${{ env.IMAGE_NAME }}-backend:${{ github.sha }},${{ env.IMAGE_NAME }}-backend:latest

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up kubectl
        uses: azure/setup-kubectl@v3
      
      - name: Configure kubectl
        run: |
          echo "${{ secrets.KUBECONFIG }}" | base64 -d > kubeconfig
          export KUBECONFIG=./kubeconfig
      
      - name: Deploy to Kubernetes
        run: |
          export KUBECONFIG=./kubeconfig
          kubectl set image deployment/frontend \
            frontend=${{ env.IMAGE_NAME }}-frontend:${{ github.sha }} \
            -n production
          kubectl set image deployment/backend \
            backend=${{ env.IMAGE_NAME }}-backend:${{ github.sha }} \
            -n production
          kubectl rollout status deployment/frontend -n production
          kubectl rollout status deployment/backend -n production
```

#### Step 5: Monitoring Setup

```yaml
# k8s/monitoring.yaml
# Prometheus ServiceMonitor
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: app-metrics
spec:
  selector:
    matchLabels:
      app: backend
  endpoints:
  - port: metrics
    interval: 30s
```

### Deliverables

1. **Source Code**:
   - Frontend application
   - Backend API
   - Dockerfiles
   - Kubernetes manifests

2. **CI/CD Pipeline**:
   - GitHub Actions workflows
   - Automated testing
   - Automated deployment

3. **Documentation**:
   - README with setup instructions
   - Architecture diagrams
   - API documentation
   - Deployment guide

4. **Monitoring**:
   - Prometheus metrics
   - Grafana dashboards
   - Alerting rules

### Challenges

1. Add blue-green deployment
2. Implement canary releases
3. Add database migrations in CI/CD
4. Set up distributed tracing
5. Implement auto-scaling

### Portfolio Value

- Demonstrates full-stack DevOps skills
- Shows CI/CD expertise
- Kubernetes experience
- Production-ready practices

## 🏗️ Project 2: Infrastructure as Code with Terraform

### Description

Provision complete cloud infrastructure using Terraform with best practices: modules, remote state, and multi-environment support.

### Architecture

```
┌─────────────────────────────────────────┐
│         Infrastructure Architecture       │
│                                           │
│  ┌─────────────────────────────────┐    │
│  │         VPC                       │    │
│  │  ┌──────────┐  ┌──────────┐     │    │
│  │  │ Public   │  │ Private  │     │    │
│  │  │ Subnets  │  │ Subnets  │     │    │
│  │  └──────────┘  └──────────┘     │    │
│  └─────────────────────────────────┘    │
│                                           │
│  ┌──────────┐  ┌──────────┐            │
│  │   EC2    │  │   RDS    │            │
│  │ Instances│  │ Database │            │
│  └──────────┘  └──────────┘            │
│                                           │
│  ┌──────────┐  ┌──────────┐            │
│  │   S3     │  │   ALB    │            │
│  │  Buckets │  │ Load Bal │            │
│  └──────────┘  └──────────┘            │
└─────────────────────────────────────────┘
```

### Technologies

- **IaC**: Terraform
- **Cloud**: AWS
- **State Management**: S3 + DynamoDB
- **Version Control**: Git
- **CI/CD**: GitHub Actions

### Implementation Steps

#### Step 1: Project Structure

```
terraform-project/
├── environments/
│   ├── dev/
│   │   └── main.tf
│   ├── staging/
│   │   └── main.tf
│   └── prod/
│       └── main.tf
├── modules/
│   ├── vpc/
│   ├── ec2/
│   ├── rds/
│   └── s3/
├── backend.tf
└── README.md
```

#### Step 2: VPC Module

```hcl
# modules/vpc/main.tf
resource "aws_vpc" "this" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = merge(var.tags, {
    Name = var.name
  })
}

resource "aws_internet_gateway" "this" {
  count  = var.create_igw ? 1 : 0
  vpc_id = aws_vpc.this.id
  
  tags = merge(var.tags, {
    Name = "${var.name}-igw"
  })
}

resource "aws_subnet" "public" {
  count             = length(var.public_subnets)
  vpc_id            = aws_vpc.this.id
  cidr_block        = var.public_subnets[count.index]
  availability_zone = var.availability_zones[count.index]
  map_public_ip_on_launch = true
  
  tags = merge(var.tags, {
    Name = "${var.name}-public-${count.index + 1}"
    Type = "Public"
  })
}

resource "aws_subnet" "private" {
  count             = length(var.private_subnets)
  vpc_id            = aws_vpc.this.id
  cidr_block        = var.private_subnets[count.index]
  availability_zone = var.availability_zones[count.index]
  
  tags = merge(var.tags, {
    Name = "${var.name}-private-${count.index + 1}"
    Type = "Private"
  })
}

resource "aws_nat_gateway" "this" {
  count = var.create_nat_gateway ? length(var.public_subnets) : 0
  
  allocation_id = aws_eip.nat[count.index].id
  subnet_id     = aws_subnet.public[count.index].id
  
  tags = merge(var.tags, {
    Name = "${var.name}-nat-${count.index + 1}"
  })
  
  depends_on = [aws_internet_gateway.this]
}

resource "aws_eip" "nat" {
  count = var.create_nat_gateway ? length(var.public_subnets) : 0
  
  domain = "vpc"
  
  tags = merge(var.tags, {
    Name = "${var.name}-nat-eip-${count.index + 1}"
  })
}
```

#### Step 3: Environment Configuration

```hcl
# environments/dev/main.tf
terraform {
  backend "s3" {
    bucket         = "terraform-state-bucket"
    key            = "environments/dev/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-state-lock"
    encrypt        = true
  }
}

module "vpc" {
  source = "../../modules/vpc"
  
  name               = "dev-vpc"
  vpc_cidr           = "10.1.0.0/16"
  availability_zones = ["us-east-1a", "us-east-1b"]
  public_subnets     = ["10.1.1.0/24", "10.1.2.0/24"]
  private_subnets    = ["10.1.11.0/24", "10.1.12.0/24"]
  
  tags = {
    Environment = "development"
    ManagedBy   = "Terraform"
  }
}

module "ec2" {
  source = "../../modules/ec2"
  
  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.public_subnet_ids
  
  instance_type = "t3.micro"
  instance_count = 2
}

module "rds" {
  source = "../../modules/rds"
  
  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnet_ids
  
  instance_class = "db.t3.micro"
  allocated_storage = 20
}
```

#### Step 4: CI/CD for Terraform

```yaml
# .github/workflows/terraform.yml
name: Terraform

on:
  push:
    paths:
      - 'terraform/**'
  pull_request:
    paths:
      - 'terraform/**'

jobs:
  terraform:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./environments/dev
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v2
        with:
          terraform_version: 1.6.0
      
      - name: Terraform Format Check
        run: terraform fmt -check
      
      - name: Terraform Init
        run: terraform init
      
      - name: Terraform Validate
        run: terraform validate
      
      - name: Terraform Plan
        run: terraform plan -out=tfplan
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      
      - name: Terraform Apply
        if: github.ref == 'refs/heads/main'
        run: terraform apply -auto-approve tfplan
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

### Deliverables

1. **Terraform Code**:
   - Reusable modules
   - Environment configurations
   - Remote state setup

2. **Documentation**:
   - Module documentation
   - Usage examples
   - Architecture diagrams

3. **CI/CD**:
   - Automated validation
   - Automated deployment
   - State management

### Challenges

1. Create custom modules
2. Implement multi-region deployment
3. Add cost estimation
4. Set up drift detection
5. Implement policy as code

## 📊 Project 3: Monitoring and Observability Stack

### Description

Set up comprehensive monitoring stack with Prometheus, Grafana, Loki, and Jaeger for complete observability.

### Architecture

```
┌─────────────────────────────────────────┐
│      Observability Stack                 │
│                                           │
│  ┌──────────┐  ┌──────────┐            │
│  │Prometheus│  │  Loki    │            │
│  │(Metrics) │  │ (Logs)   │            │
│  └────┬─────┘  └────┬─────┘            │
│       │             │                   │
│       └──────┬──────┘                   │
│              ▼                          │
│         ┌──────────┐                    │
│         │ Grafana  │                    │
│         │(Dashboards)│                  │
│         └──────────┘                    │
│                                           │
│  ┌──────────┐                           │
│  │  Jaeger  │                           │
│  │ (Traces) │                           │
│  └──────────┘                           │
└─────────────────────────────────────────┘
```

### Technologies

- **Metrics**: Prometheus
- **Visualization**: Grafana
- **Logs**: Loki
- **Traces**: Jaeger
- **Alerting**: AlertManager
- **Deployment**: Kubernetes, Helm

### Implementation Steps

#### Step 1: Prometheus Setup

```yaml
# prometheus-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
      evaluation_interval: 15s
    
    scrape_configs:
      - job_name: 'kubernetes-pods'
        kubernetes_sd_configs:
          - role: pod
        relabel_configs:
          - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
            action: keep
            regex: true
          - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
            action: replace
            target_label: __metrics_path__
            regex: (.+)
          - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
            action: replace
            regex: ([^:]+)(?::\d+)?;(\d+)
            replacement: $1:$2
            target_label: __address__
      
      - job_name: 'kubernetes-nodes'
        kubernetes_sd_configs:
          - role: node
        relabel_configs:
          - action: labelmap
            regex: __meta_kubernetes_node_label_(.+)
    
    alerting:
      alertmanagers:
        - static_configs:
            - targets:
                - alertmanager:9093
```

#### Step 2: Grafana Setup

```yaml
# grafana-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
      - name: grafana
        image: grafana/grafana:latest
        ports:
        - containerPort: 3000
        env:
        - name: GF_SECURITY_ADMIN_PASSWORD
          valueFrom:
            secretKeyRef:
              name: grafana-secret
              key: admin-password
        volumeMounts:
        - name: grafana-storage
          mountPath: /var/lib/grafana
      volumes:
      - name: grafana-storage
        persistentVolumeClaim:
          claimName: grafana-pvc
```

#### Step 3: Alerting Rules

```yaml
# prometheus-alerts.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-alerts
data:
  alerts.yml: |
    groups:
      - name: instance_alerts
        interval: 30s
        rules:
          - alert: InstanceDown
            expr: up == 0
            for: 5m
            labels:
              severity: critical
            annotations:
              summary: "Instance {{ $labels.instance }} down"
              description: "{{ $labels.instance }} has been down for more than 5 minutes."
          
          - alert: HighCPUUsage
            expr: 100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
            for: 5m
            labels:
              severity: warning
            annotations:
              summary: "High CPU usage on {{ $labels.instance }}"
              description: "CPU usage is above 80% for 5 minutes."
```

### Deliverables

1. **Monitoring Stack**:
   - Prometheus deployment
   - Grafana dashboards
   - AlertManager configuration
   - Alert rules

2. **Dashboards**:
   - System overview
   - Application metrics
   - Business metrics
   - Custom dashboards

3. **Documentation**:
   - Setup guide
   - Dashboard documentation
   - Alert runbooks

### Challenges

1. Create custom dashboards
2. Implement log correlation
3. Set up distributed tracing
4. Add anomaly detection
5. Implement alert routing

## 🔄 Project 4: GitOps Implementation

### Description

Implement complete GitOps workflow with Argo CD, managing multiple applications across environments.

### Architecture

```
┌─────────────────────────────────────────┐
│         GitOps Architecture               │
│                                           │
│  ┌──────────┐                            │
│  │   Git    │                            │
│  │ Repository│                            │
│  └────┬─────┘                            │
│       │                                  │
│       ▼                                  │
│  ┌──────────┐                            │
│  │ Argo CD  │                            │
│  │(GitOps)  │                            │
│  └────┬─────┘                            │
│       │                                  │
│       ▼                                  │
│  ┌──────────┐                            │
│  │Kubernetes│                            │
│  │  Cluster │                            │
│  └──────────┘                            │
└─────────────────────────────────────────┘
```

### Technologies

- **GitOps**: Argo CD
- **Version Control**: Git
- **Kubernetes**: Any cluster
- **Helm**: Package management

### Implementation Steps

#### Step 1: Repository Structure

```
gitops-repo/
├── apps/
│   ├── app1/
│   │   ├── base/
│   │   └── overlays/
│   │       ├── dev/
│   │       ├── staging/
│   │       └── prod/
│   └── app2/
├── infrastructure/
│   ├── monitoring/
│   ├── ingress/
│   └── secrets/
└── README.md
```

#### Step 2: Application Definitions

```yaml
# apps/app1/base/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
    spec:
      containers:
      - name: app1
        image: myregistry/app1:latest
        ports:
        - containerPort: 8080
```

#### Step 3: Argo CD Applications

```yaml
# argocd/applications.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: app1-dev
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/user/gitops-repo.git
    targetRevision: main
    path: apps/app1/overlays/dev
  destination:
    server: https://kubernetes.default.svc
    namespace: development
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

### Deliverables

1. **GitOps Repository**:
   - Application manifests
   - Environment overlays
   - Infrastructure code

2. **Argo CD Setup**:
   - Application definitions
   - Sync policies
   - Access controls

3. **Documentation**:
   - GitOps workflow
   - Application deployment guide
   - Best practices

### Challenges

1. Implement application sets
2. Add sync waves
3. Set up multi-cluster GitOps
4. Implement automated testing
5. Add compliance checks

## ✅ Project Best Practices

### 1. Documentation

- Clear README
- Architecture diagrams
- Setup instructions
- Troubleshooting guide

### 2. Version Control

- Git repository
- Meaningful commits
- Branch strategy
- Pull request reviews

### 3. CI/CD

- Automated testing
- Automated deployment
- Security scanning
- Rollback capability

### 4. Monitoring

- Metrics collection
- Log aggregation
- Alerting
- Dashboards

### 5. Security

- Secrets management
- Image scanning
- Network policies
- RBAC

## 🎯 Portfolio Tips

1. **Show Real Work**: Use real applications, not just tutorials
2. **Document Everything**: Clear documentation shows professionalism
3. **Production-Ready**: Follow best practices
4. **Demonstrate Skills**: Show variety of technologies
5. **Continuous Improvement**: Update projects regularly

## ✅ Completion Checklist

- [ ] Project 1: Multi-tier application
- [ ] Project 2: Infrastructure as Code
- [ ] Project 3: Monitoring stack
- [ ] Project 4: GitOps implementation
- [ ] All projects documented
- [ ] All projects deployed
- [ ] GitHub repositories created
- [ ] Portfolio website (optional)

---

**Next**: Explore Kubernetes projects and security projects for specialized portfolios.

**Remember**: DevOps projects demonstrate practical skills. Build real applications, follow best practices, and document everything. Good projects showcase your ability to solve real-world problems.
