# Kubernetes Interview Questions

## 🎯 Core Concepts

**Q: Explain Kubernetes architecture**
A: Control plane (API server, etcd, scheduler, controller manager) manages cluster. Worker nodes run pods via kubelet and kube-proxy.

**Q: What is a Deployment?**
A: Deployment manages pod replicas, handles updates (rolling, canary), and maintains desired state.

**Q: Explain Services**
A: Services provide stable networking for pods. ClusterIP (internal), NodePort (external), LoadBalancer (cloud), ExternalName (external).

**Q: What are ConfigMaps and Secrets?**
A: ConfigMaps store configuration data. Secrets store sensitive data. Both can be mounted as volumes or environment variables.

## 🔧 Advanced Topics

**Q: Explain Ingress**
A: Ingress provides HTTP/HTTPS routing to services. Requires Ingress controller (nginx, traefik).

**Q: What are Network Policies?**
A: Network Policies control pod-to-pod communication, enabling micro-segmentation and security.

**Q: Explain PersistentVolumes**
A: PVs provide cluster-wide storage. PVCs request storage. StorageClasses define provisioners.

## 📝 Scenario Questions

**Q: Pod is not starting. How do you troubleshoot?**
A: Check `kubectl describe pod`, view logs, check events, verify image, check resources, validate configuration.

**Q: Design a scalable Kubernetes architecture**
A: Use Deployments with HPA, implement service mesh, use cluster autoscaler, optimize resource requests/limits.

## ✅ Key Areas

- Pod lifecycle
- Deployments and scaling
- Services and networking
- Storage and volumes
- Security (RBAC, policies)
- Troubleshooting

---

**Next**: Review AWS interview questions.

