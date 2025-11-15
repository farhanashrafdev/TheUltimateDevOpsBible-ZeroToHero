# Docker Cheat Sheet - Complete Reference

## 🐳 Docker Basics

### Version and Info

```bash
# Docker version
docker --version
docker version
docker info

# System information
docker system info
docker system df                    # Disk usage
docker system events                # Real-time events
```

## 📦 Image Commands

### Listing Images

```bash
# List images
docker images
docker image ls
docker images -a                   # All images
docker images --filter "dangling=true"  # Dangling images
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
```

### Pulling Images

```bash
# Pull image
docker pull nginx
docker pull nginx:1.21
docker pull nginx:latest
docker pull nginx@sha256:abc123...  # By digest

# Pull from private registry
docker pull registry.example.com/image:tag
docker pull myregistry.io/namespace/image:tag
```

### Building Images

```bash
# Build image
docker build .
docker build -t myimage:tag .
docker build -t myimage:tag -f Dockerfile.prod .
docker build -t myimage:tag --target production .

# Build with build args
docker build --build-arg VERSION=1.0 -t myimage:1.0 .
docker build --build-arg HTTP_PROXY=http://proxy:8080 .

# Build with cache
docker build --cache-from myimage:latest -t myimage:new .

# Build without cache
docker build --no-cache -t myimage:tag .

# Build with platform
docker build --platform linux/amd64 -t myimage:tag .
docker buildx build --platform linux/amd64,linux/arm64 -t myimage:tag .
```

### Managing Images

```bash
# Inspect image
docker inspect image:tag
docker inspect --format='{{.Config.Cmd}}' image:tag

# Image history
docker history image:tag
docker history image:tag --no-trunc

# Remove image
docker rmi image:tag
docker rmi image1 image2 image3
docker rmi $(docker images -q)      # Remove all
docker rmi $(docker images -f "dangling=true" -q)  # Remove dangling

# Tag image
docker tag source:tag target:tag
docker tag myimage:1.0 myimage:latest
docker tag myimage:1.0 registry.example.com/myimage:1.0

# Save/Load image
docker save -o image.tar image:tag
docker load -i image.tar

# Export/Import image
docker export container > container.tar
docker import container.tar image:tag
```

### Pushing Images

```bash
# Push to registry
docker push image:tag
docker push registry.example.com/image:tag

# Login to registry
docker login
docker login registry.example.com
docker login -u username -p password registry.example.com

# Logout
docker logout
docker logout registry.example.com
```

## 🚀 Container Commands

### Running Containers

```bash
# Run container
docker run image:tag
docker run -d image:tag            # Detached mode
docker run -it image:tag bash      # Interactive
docker run --name mycontainer image:tag

# Run with ports
docker run -p 8080:80 image:tag
docker run -p 127.0.0.1:8080:80 image:tag
docker run -P image:tag            # Publish all ports

# Run with volumes
docker run -v /host/path:/container/path image:tag
docker run -v volume_name:/container/path image:tag
docker run --mount type=bind,source=/host,target=/container image:tag

# Run with environment variables
docker run -e VAR=value image:tag
docker run -e VAR=value --env-file .env image:tag

# Run with network
docker run --network mynetwork image:tag
docker run --network host image:tag
docker run --network none image:tag

# Run with resource limits
docker run --memory="512m" --cpus="1.0" image:tag
docker run --memory-swap="1g" image:tag

# Run with restart policy
docker run --restart always image:tag
docker run --restart unless-stopped image:tag
docker run --restart on-failure:5 image:tag

# Run with user
docker run -u 1000:1000 image:tag
docker run --user root image:tag

# Run with entrypoint override
docker run --entrypoint /bin/sh image:tag
```

### Listing Containers

```bash
# List containers
docker ps                           # Running
docker ps -a                       # All
docker ps -l                       # Latest
docker ps -q                       # IDs only
docker ps --filter "status=running"
docker ps --filter "name=nginx"
docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Status}}"
```

### Starting/Stopping Containers

```bash
# Start container
docker start container
docker start container1 container2

# Stop container
docker stop container
docker stop container1 container2
docker stop $(docker ps -q)        # Stop all

# Restart container
docker restart container

# Pause/Unpause
docker pause container
docker unpause container

# Kill container
docker kill container
docker kill -s SIGTERM container
```

### Removing Containers

```bash
# Remove container
docker rm container
docker rm container1 container2
docker rm -f container             # Force remove running
docker rm $(docker ps -aq)         # Remove all stopped
docker container prune             # Remove all stopped
```

### Container Information

```bash
# Inspect container
docker inspect container
docker inspect --format='{{.NetworkSettings.IPAddress}}' container
docker inspect --format='{{json .Config}}' container

# Container logs
docker logs container
docker logs -f container           # Follow
docker logs --tail 100 container   # Last 100 lines
docker logs --since 1h container   # Last hour
docker logs --since 2024-01-01T00:00:00 container

# Container stats
docker stats
docker stats container
docker stats --no-stream container

# Container processes
docker top container
docker top container aux

# Container diff
docker diff container               # Changed files
```

### Executing Commands

```bash
# Execute command
docker exec container ls
docker exec container cat /etc/hostname

# Interactive shell
docker exec -it container bash
docker exec -it container sh

# Execute as user
docker exec -u root container whoami
```

### Copying Files

```bash
# Copy from container
docker cp container:/path/to/file ./local-file
docker cp container:/etc/nginx/nginx.conf ./nginx.conf

# Copy to container
docker cp ./local-file container:/path/to/file
docker cp ./config.yaml container:/etc/config.yaml
```

## 🔄 Docker Compose

### Basic Commands

```bash
# Start services
docker-compose up
docker-compose up -d               # Detached
docker-compose up --build          # Rebuild images
docker-compose up service1 service2  # Specific services

# Stop services
docker-compose down
docker-compose down -v             # Remove volumes
docker-compose down --rmi all      # Remove images

# List services
docker-compose ps
docker-compose ps -a

# View logs
docker-compose logs
docker-compose logs -f             # Follow
docker-compose logs service1       # Specific service
docker-compose logs --tail=100     # Last 100 lines

# Execute command
docker-compose exec service bash
docker-compose exec service ls /app

# Run one-off command
docker-compose run service command
docker-compose run --rm service npm test
```

### Compose Management

```bash
# Start services
docker-compose start

# Stop services
docker-compose stop

# Restart services
docker-compose restart

# Scale services
docker-compose up --scale web=3

# Build images
docker-compose build
docker-compose build --no-cache

# Pull images
docker-compose pull

# Remove containers
docker-compose rm

# Pause/Unpause
docker-compose pause
docker-compose unpause
```

## 🌐 Network Commands

### Network Management

```bash
# List networks
docker network ls
docker network inspect network

# Create network
docker network create mynetwork
docker network create --driver bridge mynetwork
docker network create --subnet=172.20.0.0/16 mynetwork

# Remove network
docker network rm network
docker network prune              # Remove unused

# Connect container to network
docker network connect network container
docker network disconnect network container
```

### Network Types

```bash
# Bridge network (default)
docker network create --driver bridge mybridge

# Host network
docker run --network host image:tag

# None network (isolated)
docker run --network none image:tag

# Overlay network (Swarm)
docker network create --driver overlay myoverlay
```

## 💾 Volume Commands

### Volume Management

```bash
# List volumes
docker volume ls
docker volume inspect volume

# Create volume
docker volume create myvolume
docker volume create --driver local myvolume

# Remove volume
docker volume rm volume
docker volume prune               # Remove unused

# Inspect volume
docker volume inspect myvolume
```

### Volume Types

```bash
# Named volume
docker run -v myvolume:/data image:tag

# Bind mount
docker run -v /host/path:/container/path image:tag

# Anonymous volume
docker run -v /data image:tag

# Tmpfs mount
docker run --tmpfs /tmp image:tag
```

## 🏷️ Tagging and Labels

```bash
# Tag image
docker tag source:tag target:tag
docker tag myimage:1.0 myimage:latest
docker tag myimage:1.0 registry.com/myimage:1.0

# Add labels
docker build --label "version=1.0" -t myimage:tag .
docker run --label "env=production" image:tag
```

## 🔍 Search and Filter

```bash
# Search images
docker search nginx
docker search nginx --limit 10

# Filter containers
docker ps --filter "name=nginx"
docker ps --filter "status=running"
docker ps --filter "ancestor=nginx:1.21"

# Filter images
docker images --filter "dangling=true"
docker images --filter "before=image:tag"
docker images --filter "since=image:tag"
```

## 🧹 Cleanup Commands

```bash
# Remove unused containers
docker container prune

# Remove unused images
docker image prune
docker image prune -a              # All unused

# Remove unused volumes
docker volume prune

# Remove unused networks
docker network prune

# Remove everything
docker system prune
docker system prune -a              # All unused
docker system prune -a --volumes   # Include volumes

# Remove by filter
docker images prune --filter "until=24h"
```

## 🔐 Security Commands

```bash
# Scan image for vulnerabilities
docker scan image:tag
docker scan --file Dockerfile image:tag

# Run as non-root
docker run -u 1000:1000 image:tag

# Read-only root filesystem
docker run --read-only image:tag

# Security options
docker run --security-opt seccomp=unconfined image:tag
docker run --security-opt apparmor=profile image:tag
```

## 📊 Monitoring and Debugging

```bash
# Container stats
docker stats
docker stats --no-stream
docker stats container1 container2

# Events
docker events
docker events --filter "container=container"
docker events --since "2024-01-01"

# System information
docker info
docker system df
docker system events

# Inspect
docker inspect container
docker inspect image:tag
docker inspect network
docker inspect volume
```

## 🛠️ Advanced Commands

### Buildx (Multi-platform)

```bash
# Create builder
docker buildx create --name mybuilder
docker buildx use mybuilder

# Build for multiple platforms
docker buildx build --platform linux/amd64,linux/arm64 -t image:tag .

# Inspect builder
docker buildx inspect
docker buildx inspect --bootstrap
```

### Dockerfile Best Practices

```dockerfile
# Multi-stage build
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html

# Use specific tags
FROM nginx:1.21-alpine

# Minimize layers
RUN apt-get update && \
    apt-get install -y package && \
    rm -rf /var/lib/apt/lists/*

# Use .dockerignore
# node_modules
# .git
# *.log
```

## 📝 Common Patterns

### Development Container

```bash
docker run -it --rm \
  -v $(pwd):/app \
  -w /app \
  -p 3000:3000 \
  node:18 \
  npm start
```

### Production Container

```bash
docker run -d \
  --name app \
  --restart unless-stopped \
  -p 80:8080 \
  -e NODE_ENV=production \
  -v app-data:/data \
  app:latest
```

### Debug Container

```bash
docker run -it --rm \
  --entrypoint /bin/sh \
  image:tag
```

## 🚨 Troubleshooting

```bash
# Container won't start
docker logs container
docker inspect container
docker events --filter "container=container"

# Image build fails
docker build --progress=plain -t image:tag .
docker build --no-cache -t image:tag .

# Network issues
docker network inspect network
docker exec container ping host

# Volume issues
docker volume inspect volume
docker exec container ls -la /path
```

## 📚 Quick Reference

### Common Aliases

```bash
alias d='docker'
alias dc='docker-compose'
alias dps='docker ps'
alias dimg='docker images'
alias dexec='docker exec -it'
alias dlog='docker logs -f'
```

### Resource Limits

```bash
# Memory
--memory="512m"
--memory-swap="1g"
--oom-kill-disable

# CPU
--cpus="1.5"
--cpuset-cpus="0,1"
--cpu-shares=1024

# I/O
--device-read-bps /dev/sda:1mb
--device-write-bps /dev/sda:1mb
```

---

**Remember**: Docker commands are powerful. Always verify before removing resources. Use `--dry-run` or test in non-production first.
