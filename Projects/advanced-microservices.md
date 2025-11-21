# Project: Microservices with Service Mesh (Advanced)

## 🎯 Objective
Deploy a microservices architecture with observability, traffic splitting, and mTLS using Istio.

## 🛠️ Tech Stack
- **App**: Google Online Boutique (Demo App)
- **Cluster**: EKS / GKE / AKS
- **Mesh**: Istio / Linkerd
- **Observability**: Prometheus, Grafana, Jaeger

## 📝 Requirements

1. **Infrastructure**:
   - Provision K8s cluster with Terraform.
   - Install Istio via Helm.

2. **Deployment**:
   - Deploy microservices (10+ services).
   - Inject Envoy sidecars.

3. **Traffic Management**:
   - Implement Canary Deployment (90% v1, 10% v2).
   - Implement Circuit Breaking.

4. **Observability**:
   - Visualize service graph (Kiali).
   - Trace requests (Jaeger).
   - Monitor metrics (Grafana).

## 🚀 Steps

### 1. Cluster Setup
- Provision managed K8s.
- Install Istio.

### 2. Deploy & Mesh
- Deploy app.
- Enable sidecar injection.
- Verify mTLS is active.

### 3. Traffic Engineering
- Create `VirtualService` and `DestinationRule`.
- Shift traffic gradually.
- Test failure scenarios (Circuit Breaker).

## 🏆 Definition of Done
- [ ] All services communicate via mTLS.
- [ ] Can perform canary rollout without downtime.
- [ ] Service graph visible in Kiali.
- [ ] Traces show full request path.

---

**Next**: [Expert: Platform Engineering](./expert-platform.md).
