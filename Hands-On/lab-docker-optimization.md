# Lab: Docker Image Optimization

## 🎯 Scenario
Your Docker image is 1GB. Deployments are slow. Reduce it to < 100MB.

## 🛠️ Setup (The Bloated Image)
```dockerfile
FROM ubuntu:latest
RUN apt-get update && apt-get install -y python3 python3-pip build-essential
COPY . /app
WORKDIR /app
RUN pip3 install -r requirements.txt
CMD ["python3", "app.py"]
```

## 🕵️ Optimization Steps

### 1. Use a Smaller Base Image
Switch `ubuntu:latest` to `python:3.9-slim` or `alpine`.

### 2. Multi-Stage Build
Separate build tools from runtime.

```dockerfile
# Stage 1: Build
FROM python:3.9-slim as builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user -r requirements.txt

# Stage 2: Runtime
FROM python:3.9-slim
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY . .
ENV PATH=/root/.local/bin:$PATH
CMD ["python", "app.py"]
```

### 3. .dockerignore
Exclude `.git`, `venv`, `tests`.

```text
.git
venv
__pycache__
*.md
```

### 4. Combine RUN Instructions
Reduce layers (less important now, but good practice).

## ✅ Success Criteria
- [ ] Image size reduced significantly (e.g., 1GB -> 80MB).
- [ ] Application still runs correctly.
- [ ] Build time is reasonable.
