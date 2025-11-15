# DevOps Labs

## 🎯 Practical Hands-On Exercises

### Lab 1: Linux Fundamentals
**Objective**: Master Linux command line

**Tasks**:
1. Navigate file system
2. Create and manage files
3. Use text processing tools
4. Manage processes
5. Configure networking

**Commands to Practice**:
```bash
# File operations
ls -lah
find /var/log -name "*.log" -mtime -7
grep -r "error" /var/log

# Process management
ps aux | grep nginx
kill -9 PID
systemctl status nginx

# Networking
ip addr show
netstat -tulpn
curl -I https://example.com
```

### Lab 2: Docker Basics
**Objective**: Containerize an application

**Tasks**:
1. Create Dockerfile
2. Build image
3. Run container
4. Use volumes
5. Network containers

**Dockerfile Example**:
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

**Commands**:
```bash
docker build -t myapp .
docker run -d -p 3000:3000 myapp
docker exec -it container_id sh
```

### Lab 3: CI/CD Pipeline
**Objective**: Build CI/CD pipeline

**Tasks**:
1. Set up GitHub Actions
2. Configure build steps
3. Add tests
4. Deploy to staging

**Pipeline Example**:
```yaml
name: CI/CD
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build
        run: docker build -t app .
      - name: Test
        run: docker run app npm test
```

### Lab 4: Infrastructure as Code
**Objective**: Provision infrastructure with Terraform

**Tasks**:
1. Write Terraform configuration
2. Initialize Terraform
3. Plan changes
4. Apply infrastructure

**Terraform Example**:
```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
}
```

### Lab 5: Kubernetes Basics
**Objective**: Deploy application on Kubernetes

**Tasks**:
1. Create deployment
2. Expose service
3. Scale application
4. View logs

**Deployment Example**:
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

## ✅ Completion Checklist

- [ ] Completed all labs
- [ ] Documented learnings
- [ ] Troubleshot issues
- [ ] Applied best practices

---

**Next**: Complete CI/CD labs.

