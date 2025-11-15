# Troubleshooting Guide

## 🎯 Systematic Approach

### 1. Gather Information
```bash
# Check logs
kubectl logs pod-name
journalctl -u service-name

# Check resources
kubectl top pods
kubectl describe pod pod-name

# Check events
kubectl get events
```

### 2. Identify Symptoms
- What is failing?
- When did it start?
- What changed recently?
- Who is affected?

### 3. Hypothesis
- Form theories
- Test hypotheses
- Narrow down causes

### 4. Resolution
- Apply fix
- Verify solution
- Document learnings

## 🔍 Common Issues

### Pod Not Starting
```bash
# Check pod status
kubectl get pods
kubectl describe pod pod-name

# Check logs
kubectl logs pod-name

# Check events
kubectl get events --field-selector involvedObject.name=pod-name
```

### High CPU/Memory
```bash
# Check resource usage
kubectl top pods
kubectl top nodes

# Check limits
kubectl describe pod pod-name | grep Limits
```

### Network Issues
```bash
# Test connectivity
kubectl exec pod-name -- ping service-name

# Check services
kubectl get svc
kubectl describe svc service-name
```

## 📝 Debugging Tools

- **kubectl**: Kubernetes CLI
- **kubectl debug**: Debug containers
- **tcpdump**: Network analysis
- **strace**: System calls
- **perf**: Performance analysis

## ✅ Best Practices

- Start with logs
- Use systematic approach
- Document findings
- Learn from incidents
- Build runbooks
- Practice regularly

---

**Next**: Explore Platform Engineering concepts.

