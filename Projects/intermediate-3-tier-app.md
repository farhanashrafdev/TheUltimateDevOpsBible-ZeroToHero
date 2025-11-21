# Project: 3-Tier Containerized Application (Intermediate)

## 🎯 Objective
Deploy a 3-tier application (Frontend, Backend, Database) using Docker and Docker Compose, then migrate to Kubernetes.

## 🛠️ Tech Stack
- **App**: React (Front), Python/Flask (Back), PostgreSQL (DB)
- **Container**: Docker, Docker Compose
- **Orchestration**: Kubernetes (Minikube or EKS)
- **Reverse Proxy**: Nginx

## 📝 Requirements

1. **Dockerization**:
   - Write `Dockerfile` for Frontend and Backend.
   - Optimize images (multi-stage builds).
   - Write `docker-compose.yml` for local dev.

2. **Kubernetes Deployment**:
   - Write K8s manifests (Deployment, Service, ConfigMap, Secret).
   - Use `StatefulSet` for PostgreSQL.
   - Use `Ingress` for routing traffic.

3. **Networking**:
   - Frontend talks to Backend via Service DNS.
   - Backend talks to DB via Secret credentials.

## 🚀 Steps

### 1. Local Development
- Get a sample 3-tier app.
- Containerize it.
- Run with `docker compose up`.

### 2. Kubernetes Migration
- Translate Compose to K8s manifests.
- Deploy to local cluster (Kind/Minikube).
- Verify pod-to-pod communication.

### 3. Helm Chart
- Package the application as a Helm chart.
- Parameterize values (image tag, replicas).

## 🏆 Definition of Done
- [ ] App runs locally with Docker Compose.
- [ ] App runs on Kubernetes.
- [ ] Data persists after DB pod restart (PVC).
- [ ] Ingress routes traffic correctly.
- [ ] Helm chart installs the app with one command.

---

**Next**: [Advanced: Microservices](./advanced-microservices.md).
