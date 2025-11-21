# DevOps Hands-On Labs: Complete Practical Guide

## 🎯 Introduction

This comprehensive guide provides hands-on labs to practice DevOps skills. Each lab includes objectives, step-by-step instructions, and exercises.

## 📚 Lab Structure

Each lab includes:
- **Objective**: What you'll learn
- **Prerequisites**: Required knowledge/tools
- **Tasks**: Step-by-step exercises
- **Commands**: Code snippets
- **Verification**: How to verify success
- **Challenges**: Advanced exercises

## 🐧 Lab 1: Linux Fundamentals

### Objective
Master Linux command line for DevOps tasks.

### Prerequisites
- Linux VM or WSL
- Basic terminal knowledge

### Tasks

#### Task 1: File System Navigation
```bash
# Navigate directories
cd /var/log
pwd
ls -lah

# Find files
find /var/log -name "*.log" -mtime -7
find /home -type f -size +100M

# File operations
mkdir -p /tmp/lab1/{dir1,dir2,dir3}
touch /tmp/lab1/file{1..5}.txt
cp /tmp/lab1/file1.txt /tmp/lab1/dir1/
mv /tmp/lab1/file2.txt /tmp/lab1/dir2/
rm /tmp/lab1/file3.txt
```

#### Task 2: Text Processing
```bash
# View files
cat /etc/passwd
head -n 20 /var/log/syslog
tail -f /var/log/syslog

# Search and filter
grep -r "error" /var/log/
grep -i "ERROR" /var/log/syslog | head -10
grep -v "^#" /etc/ssh/sshd_config

# Text manipulation
cut -d: -f1 /etc/passwd
awk '{print $1, $3}' /etc/passwd
sed 's/old/new/g' file.txt
sort /etc/passwd | uniq
```

#### Task 3: Process Management
```bash
# View processes
ps aux
ps aux | grep nginx
top
htop

# Process control
kill -9 PID
pkill nginx
killall nginx

# Background jobs
nohup command &
jobs
fg %1
bg %1
```

#### Task 4: System Information
```bash
# System info
uname -a
hostname
uptime
free -h
df -h
du -sh /var/log/*

# Network
ip addr show
ip route show
netstat -tulpn
ss -tulpn
curl -I https://example.com
wget https://example.com/file.txt
```

#### Task 5: Permissions
```bash
# View permissions
ls -l file.txt
stat file.txt

# Change permissions
chmod 755 script.sh
chmod +x script.sh
chmod u+x,g-w,o-r file.txt

# Change ownership
chown user:group file.txt
chown -R user:group /path/to/dir
```

### Verification
```bash
# Verify tasks completed
ls -lah /tmp/lab1/
grep -c "error" /var/log/syslog
ps aux | grep -c nginx
```

### Challenges
1. Create a script that finds all files larger than 100MB
2. Parse log file and count errors by type
3. Monitor system resources and alert on high usage

## 🐳 Lab 2: Docker Basics

### Objective
Containerize an application and understand Docker fundamentals.

### Prerequisites
- Docker installed
- Basic Linux knowledge

### Tasks

#### Task 1: Docker Basics
```bash
# Check Docker
docker --version
docker info

# Images
docker images
docker pull nginx:latest
docker rmi nginx:latest

# Containers
docker ps
docker ps -a
docker run -d --name nginx nginx:latest
docker stop nginx
docker start nginx
docker rm nginx
```

#### Task 2: Create Dockerfile
```dockerfile
# Create Dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .

EXPOSE 3000

USER node

CMD ["node", "server.js"]
```

```bash
# Build image
docker build -t myapp:latest .

# Run container
docker run -d -p 3000:3000 --name myapp myapp:latest

# View logs
docker logs myapp
docker logs -f myapp

# Execute commands
docker exec -it myapp sh
docker exec myapp node --version
```

#### Task 3: Docker Compose
```yaml
# docker-compose.yml
version: '3.8'
services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    depends_on:
      - db
  
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: password
      POSTGRES_DB: myapp
    volumes:
      - db_data:/var/lib/postgresql/data

volumes:
  db_data:
```

```bash
# Start services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop services
docker-compose down

# Remove volumes
docker-compose down -v
```

#### Task 4: Volumes and Networking
```bash
# Create volume
docker volume create myvolume

# Use volume
docker run -d -v myvolume:/data nginx

# Bind mount
docker run -d -v /host/path:/container/path nginx

# Network
docker network create mynetwork
docker run -d --network mynetwork --name app1 nginx
docker run -d --network mynetwork --name app2 nginx
```

### Verification
```bash
# Verify containers running
docker ps

# Test application
curl http://localhost:3000

# Check volumes
docker volume ls
```

### Challenges
1. Create multi-stage Dockerfile
2. Set up Docker Compose with 3 services
3. Implement health checks

## 🔄 Lab 3: CI/CD Pipeline

### Objective
Build a complete CI/CD pipeline with GitHub Actions.

### Prerequisites
- GitHub account
- Docker installed
- Basic Git knowledge

### Tasks

#### Task 1: Basic CI Pipeline
```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

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
        run: npm ci
      
      - name: Run tests
        run: npm test
      
      - name: Run linting
        run: npm run lint
```

#### Task 2: Build and Push Image
```yaml
# .github/workflows/build.yml
name: Build and Push

on:
  push:
    branches: [ main ]

jobs:
  build:
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
      
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: user/app:${{ github.sha }}
```

#### Task 3: Security Scanning
```yaml
# .github/workflows/security.yml
name: Security Scan

on: [push, pull_request]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          format: 'sarif'
          output: 'trivy-results.sarif'
      
      - name: Upload Trivy results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'
```

#### Task 4: Deployment
```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to staging
        run: |
          echo "Deploying to staging..."
          # Add deployment commands
      
      - name: Run smoke tests
        run: |
          curl -f http://staging.example.com/health || exit 1
      
      - name: Deploy to production
        if: github.ref == 'refs/heads/main'
        run: |
          echo "Deploying to production..."
          # Add deployment commands
```

### Verification
- Check GitHub Actions runs
- Verify images pushed to registry
- Test deployed application

### Challenges
1. Add automated testing
2. Implement canary deployment
3. Add rollback mechanism

## ☸️ Lab 4: Kubernetes Basics

### Objective
Deploy and manage applications on Kubernetes.

### Prerequisites
- Kubernetes cluster (minikube, kind, or cloud)
- kubectl installed

### Tasks

#### Task 1: Pod Basics
```yaml
# pod.yaml
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

```bash
# Create pod
kubectl apply -f pod.yaml

# View pods
kubectl get pods
kubectl get pods -o wide

# Describe pod
kubectl describe pod nginx-pod

# View logs
kubectl logs nginx-pod

# Execute commands
kubectl exec -it nginx-pod -- sh

# Delete pod
kubectl delete pod nginx-pod
```

#### Task 2: Deployments
```yaml
# deployment.yaml
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
        ports:
        - containerPort: 80
```

```bash
# Create deployment
kubectl apply -f deployment.yaml

# View deployment
kubectl get deployments
kubectl describe deployment nginx-deployment

# Scale deployment
kubectl scale deployment nginx-deployment --replicas=5

# Update deployment
kubectl set image deployment/nginx-deployment nginx=nginx:1.22

# Rollout status
kubectl rollout status deployment/nginx-deployment

# Rollback
kubectl rollout undo deployment/nginx-deployment
```

#### Task 3: Services
```yaml
# service.yaml
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

```bash
# Create service
kubectl apply -f service.yaml

# View services
kubectl get svc
kubectl describe svc nginx-service

# Test connectivity
kubectl run -it --rm debug --image=busybox --restart=Never -- wget -O- http://nginx-service:80
```

#### Task 4: ConfigMaps and Secrets
```bash
# Create ConfigMap
kubectl create configmap app-config --from-literal=key1=value1 --from-literal=key2=value2

# Create Secret
kubectl create secret generic app-secret --from-literal=password=secret123

# View
kubectl get configmap app-config -o yaml
kubectl get secret app-secret -o yaml
```

### Verification
```bash
# Verify deployments
kubectl get all

# Test services
curl http://nginx-service
```

### Challenges
1. Create StatefulSet
2. Implement HPA
3. Set up Ingress

## ✅ Completion Checklist

- [ ] Completed all labs
- [ ] Documented learnings
- [ ] Troubleshot issues
- [ ] Applied best practices
- [ ] Completed challenges

---

**Next**: Complete CI/CD labs and Kubernetes labs for advanced practice.
