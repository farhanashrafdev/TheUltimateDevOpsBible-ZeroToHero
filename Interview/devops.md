# DevOps Interview Questions

## 🎯 Introduction

This guide covers comprehensive DevOps interview questions from fundamentals to advanced topics, including scenario-based questions commonly asked at FAANG/MAANG companies.

## 📚 Fundamentals

### What is DevOps?

**Q: Define DevOps and its core principles.**

**A:** DevOps is a cultural and technical movement that combines development (Dev) and operations (Ops) to enable faster, more reliable software delivery. Core principles:

- **Culture**: Break down silos, shared responsibility
- **Automation**: CI/CD, IaC, configuration management
- **Lean**: Eliminate waste, continuous improvement
- **Measurement**: Data-driven decisions, monitoring
- **Sharing**: Knowledge sharing, blameless postmortems

**Q: Explain the difference between DevOps, SRE, and Platform Engineering.**

**A:**
- **DevOps**: Cultural movement focused on collaboration and automation
- **SRE**: Google's implementation of DevOps with focus on reliability (SLOs, error budgets)
- **Platform Engineering**: Building internal platforms to abstract infrastructure complexity

### CI/CD

**Q: Explain CI/CD and its benefits.**

**A:** 
- **CI (Continuous Integration)**: Automatically build, test, and validate code on every commit
- **CD (Continuous Delivery)**: Automatically prepare releases for deployment
- **CD (Continuous Deployment)**: Automatically deploy to production

Benefits: Faster feedback, reduced risk, consistent releases, improved quality

**Q: What's the difference between Continuous Delivery and Continuous Deployment?**

**A:**
- **Continuous Delivery**: Code is always deployable, but requires manual approval
- **Continuous Deployment**: Every passing change automatically deploys to production

## 🐳 Docker & Containers

**Q: Explain the difference between Docker image and container.**

**A:**
- **Image**: Read-only template containing application and dependencies
- **Container**: Running instance of an image with its own filesystem, network, process space

**Q: How do Docker layers work?**

**A:** Each Dockerfile instruction creates a layer. Layers are:
- Cached and reusable
- Stacked on top of each other
- Read-only (except the top writable layer)
- Shared between images using same base

**Q: What is a multi-stage build and why use it?**

**A:** Multi-stage builds use multiple FROM statements to:
- Separate build and runtime environments
- Reduce final image size (no build tools)
- Improve security (fewer attack vectors)

```dockerfile
# Build stage
FROM node:18 AS builder
WORKDIR /app
COPY . .
RUN npm ci && npm run build

# Runtime stage
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
```

**Q: How would you troubleshoot a container that keeps crashing?**

**A:**
1. Check logs: `docker logs <container>`
2. Check exit code: `docker inspect --format='{{.State.ExitCode}}'`
3. Run interactively: `docker run -it <image> sh`
4. Check resource limits: Memory/CPU constraints
5. Verify health checks and startup probes
6. Check for missing environment variables or configs

## ☸️ Kubernetes

**Q: Explain Kubernetes architecture.**

**A:**
- **Control Plane**: API Server, etcd, Scheduler, Controller Manager
- **Worker Nodes**: kubelet, kube-proxy, Container Runtime
- **API Server**: Front-end, validates requests, updates etcd
- **etcd**: Distributed key-value store for cluster state
- **Scheduler**: Assigns pods to nodes
- **kubelet**: Node agent, manages pod lifecycle

**Q: What's the difference between Deployment and StatefulSet?**

**A:**
- **Deployment**: Stateless apps, pods are interchangeable, random names
- **StatefulSet**: Stateful apps, stable pod identity, ordered deployment, persistent storage per pod

**Q: Explain Kubernetes networking model.**

**A:**
- Every pod gets unique IP
- Pods can communicate without NAT
- Services abstract pod IPs (ClusterIP, NodePort, LoadBalancer)
- Ingress for external HTTP routing

**Q: How would you debug a pod stuck in CrashLoopBackOff?**

**A:**
```bash
# Check pod events
kubectl describe pod <pod-name>

# Check logs
kubectl logs <pod-name> --previous

# Check resources
kubectl top pod <pod-name>

# Exec into pod (if possible)
kubectl exec -it <pod-name> -- sh

# Check YAML configuration
kubectl get pod <pod-name> -o yaml
```

## 🔧 Infrastructure as Code

**Q: Explain Terraform state and why it's important.**

**A:** State tracks:
- Resource IDs mapped to configuration
- Metadata and dependencies
- Enables plan/apply operations

Best practices:
- Remote state (S3, GCS)
- State locking (DynamoDB)
- Never edit state manually
- Use workspaces for environments

**Q: How do you handle secrets in Terraform?**

**A:**
- Never commit secrets to version control
- Use environment variables or `-var` flags
- Integrate with Vault or AWS Secrets Manager
- Mark sensitive outputs: `sensitive = true`
- Use SOPS for encrypted tfvars

**Q: Explain Terraform modules.**

**A:** Modules are reusable Terraform configurations:
- Encapsulate related resources
- Accept input variables
- Expose outputs
- Version controlled
- Enable DRY infrastructure

## 📊 Monitoring & Observability

**Q: What are the three pillars of observability?**

**A:**
1. **Metrics**: Numerical data over time (Prometheus)
2. **Logs**: Detailed event records (Loki, ELK)
3. **Traces**: Request flow across services (Jaeger, Zipkin)

**Q: Explain the difference between monitoring and observability.**

**A:**
- **Monitoring**: Collecting known metrics, alerts on thresholds
- **Observability**: Understanding system state from outputs, debugging unknown issues

**Q: How would you design alerting for a microservices architecture?**

**A:**
- Alert on symptoms (user impact), not causes
- Use SLO-based alerting
- Implement alert hierarchy (page/ticket/log)
- Avoid alert fatigue with proper thresholds
- Include runbooks with alerts
- Use multi-window burn rate alerts

## 🔐 Security Scenarios

**Q: How do you handle secrets in Kubernetes?**

**A:**
- Kubernetes Secrets (base64, not encrypted by default)
- Enable encryption at rest
- External Secrets Operator with Vault/AWS Secrets Manager
- Sealed Secrets for GitOps
- SOPS with age/KMS

**Q: Explain the principle of least privilege.**

**A:** Grant only minimum permissions required:
- Time-limited access (JIT)
- Role-based access control
- Regular access reviews
- Separate service accounts per component

## 🎯 Scenario Questions

**Q: A production deployment caused errors. How do you handle it?**

**A:**
1. **Immediate**: Roll back deployment
2. **Communicate**: Notify stakeholders
3. **Investigate**: Check logs, metrics, recent changes
4. **Root cause**: Analyze what went wrong
5. **Fix**: Implement proper fix
6. **Prevent**: Add tests, improve CI/CD gates
7. **Document**: Blameless postmortem

**Q: Design a zero-downtime deployment strategy.**

**A:**
- **Blue-Green**: Two identical environments, switch traffic
- **Canary**: Gradual rollout to percentage of users
- **Rolling**: Replace pods gradually

Key considerations:
- Backward-compatible database changes
- Health checks before receiving traffic
- Quick rollback capability
- Feature flags for new functionality

**Q: How would you reduce deployment time from 30 minutes to 5 minutes?**

**A:**
1. Parallelize test execution
2. Use faster CI runners (self-hosted, larger instances)
3. Implement efficient caching (dependencies, Docker layers)
4. Optimize Docker builds (multi-stage, minimal base images)
5. Skip unnecessary steps (incremental builds)
6. Use test selection (only affected tests)

**Q: A critical service is experiencing 50% error rate. Walk through your debugging process.**

**A:**
1. **Assess scope**: Which endpoints? Which users?
2. **Check recent changes**: Deployments, config changes
3. **Review metrics**: CPU, memory, connections, queue depth
4. **Check dependencies**: Database, external APIs, DNS
5. **Analyze logs**: Error patterns, stack traces
6. **Trace requests**: Identify where failures occur
7. **Mitigate**: Scale, rollback, failover
8. **Document**: Timeline, actions, recovery

## ✅ Quick Reference

### Commands to Know

```bash
# Docker
docker build -t app:v1 .
docker run -d -p 8080:80 app:v1
docker exec -it <container> sh
docker logs -f <container>

# Kubernetes
kubectl get pods -A
kubectl describe pod <pod>
kubectl logs -f <pod>
kubectl exec -it <pod> -- sh
kubectl rollout restart deployment/<name>
kubectl rollout undo deployment/<name>

# Terraform
terraform init
terraform plan -out=plan.tfplan
terraform apply plan.tfplan
terraform state list
terraform import <resource> <id>
```

---

**Next**: Review [Kubernetes Interview](./kubernetes.md) questions.

