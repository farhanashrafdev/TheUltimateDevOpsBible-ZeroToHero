# Docker Fundamentals

## 🎯 Introduction

Docker is a platform for developing, shipping, and running applications in containers. Containers package applications with all dependencies, ensuring consistency across environments.

## 📚 Core Concepts

### Container vs Virtual Machine

```
Virtual Machine:
┌─────────────────────────────────┐
│        Application              │
│        Bins/Libs                │
│        Guest OS                 │
│    ┌──────────────────────┐    │
│    │   Hypervisor         │    │
│    └──────────────────────┘    │
│        Host OS                  │
│        Infrastructure           │
└─────────────────────────────────┘

Container:
┌─────────────────────────────────┐
│        Application              │
│        Bins/Libs                │
│    ┌──────────────────────┐    │
│    │   Docker Engine      │    │
│    └──────────────────────┘    │
│        Host OS                  │
│        Infrastructure           │
└─────────────────────────────────┘
```

## 🐳 Installation

### Ubuntu/Debian

```bash
# Update packages
sudo apt update

# Install prerequisites
sudo apt install -y ca-certificates curl gnupg lsb-release

# Add Docker's official GPG key
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
    sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Add Docker repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Start Docker
sudo systemctl start docker
sudo systemctl enable docker

# Add user to docker group
sudo usermod -aG docker $USER
```

### Verify Installation

```bash
docker --version
docker run hello-world
```

## 🔧 Basic Commands

### Images

```bash
# Pull image
docker pull nginx:latest
docker pull ubuntu:20.04

# List images
docker images
docker image ls

# Remove image
docker rmi nginx
docker image rm nginx

# Search images
docker search nginx

# Inspect image
docker inspect nginx
```

### Containers

```bash
# Run container
docker run nginx
docker run -d nginx                    # Detached mode
docker run -it ubuntu bash             # Interactive
docker run -p 8080:80 nginx            # Port mapping
docker run -v /host:/container nginx   # Volume mount

# List containers
docker ps                               # Running
docker ps -a                            # All

# Stop container
docker stop container_id

# Start container
docker start container_id

# Remove container
docker rm container_id
docker rm -f container_id               # Force remove

# Execute command in container
docker exec -it container_id bash

# View logs
docker logs container_id
docker logs -f container_id             # Follow logs

# Container stats
docker stats
docker stats container_id
```

## 📝 Dockerfile

### Basic Dockerfile

```dockerfile
# Use base image
FROM ubuntu:20.04

# Set working directory
WORKDIR /app

# Set environment variables
ENV DEBIAN_FRONTEND=noninteractive

# Install dependencies
RUN apt update && \
    apt install -y nginx && \
    apt clean && \
    rm -rf /var/lib/apt/lists/*

# Copy files
COPY index.html /var/www/html/

# Expose port
EXPOSE 80

# Set command
CMD ["nginx", "-g", "daemon off;"]
```

### Multi-stage Build

```dockerfile
# Build stage
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Best Practices

1. **Use specific tags**: `node:18` not `node:latest`
2. **Minimize layers**: Combine RUN commands
3. **Use .dockerignore**: Exclude unnecessary files
4. **Multi-stage builds**: Reduce image size
5. **Non-root user**: Security best practice
6. **Health checks**: Monitor container health
7. **Cache layers**: Optimize build speed

## 🏗️ Docker Compose

### docker-compose.yml

```yaml
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html
    depends_on:
      - app
  
  app:
    build: .
    environment:
      - DATABASE_URL=postgresql://db:5432/mydb
    depends_on:
      - db
  
  db:
    image: postgres:14
    environment:
      - POSTGRES_DB=mydb
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - db_data:/var/lib/postgresql/data

volumes:
  db_data:
```

### Compose Commands

```bash
# Start services
docker-compose up
docker-compose up -d              # Detached

# Stop services
docker-compose down

# Build and start
docker-compose up --build

# View logs
docker-compose logs
docker-compose logs -f web

# Execute command
docker-compose exec web bash
```

## 🔐 Security Best Practices

1. **Use official images**: From trusted sources
2. **Scan images**: For vulnerabilities
3. **Non-root user**: Don't run as root
4. **Minimal base images**: Alpine Linux
5. **Secrets management**: Don't hardcode secrets
6. **Regular updates**: Keep images updated
7. **Network policies**: Limit container communication

## 📊 Docker Networking

### Network Types

```bash
# Bridge (default)
docker network create mynetwork
docker run --network mynetwork nginx

# Host
docker run --network host nginx

# None
docker run --network none nginx
```

### Network Commands

```bash
# List networks
docker network ls

# Inspect network
docker network inspect bridge

# Connect container to network
docker network connect mynetwork container_id

# Disconnect
docker network disconnect mynetwork container_id
```

## 💾 Volumes

### Volume Types

```bash
# Named volume
docker volume create myvolume
docker run -v myvolume:/data nginx

# Bind mount
docker run -v /host/path:/container/path nginx

# Anonymous volume
docker run -v /data nginx
```

### Volume Commands

```bash
# List volumes
docker volume ls

# Inspect volume
docker volume inspect myvolume

# Remove volume
docker volume rm myvolume
```

## 🎯 Common Use Cases

### Development Environment

```yaml
version: '3.8'
services:
  app:
    build: .
    volumes:
      - .:/app
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
```

### Production Deployment

```yaml
version: '3.8'
services:
  app:
    image: myapp:latest
    restart: always
    ports:
      - "80:3000"
    environment:
      - NODE_ENV=production
```

## ✅ Mastery Checklist

- [ ] Install and configure Docker
- [ ] Build Docker images
- [ ] Run and manage containers
- [ ] Write Dockerfiles
- [ ] Use Docker Compose
- [ ] Manage volumes and networks
- [ ] Understand security best practices
- [ ] Optimize image size
- [ ] Debug container issues
- [ ] Deploy containers to production

---

**Next**: Learn about container orchestration with Kubernetes.

