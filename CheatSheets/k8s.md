# Kubernetes Cheat Sheet

## 📚 Essential Commands

### Pods
```bash
kubectl get pods      # List pods
kubectl describe pod  # Describe pod
kubectl logs pod      # View logs
kubectl exec -it pod -- sh  # Execute
kubectl delete pod    # Delete pod
```

### Deployments
```bash
kubectl get deployments  # List deployments
kubectl scale deployment --replicas=5  # Scale
kubectl rollout status deployment  # Status
kubectl rollout undo deployment   # Rollback
```

### Services
```bash
kubectl get services  # List services
kubectl describe svc  # Describe service
kubectl expose deployment  # Expose service
```

### General
```bash
kubectl apply -f file.yaml  # Apply manifest
kubectl get all      # Get all resources
kubectl get events   # View events
kubectl config get-contexts  # List contexts
```

---

**Quick Reference**: Essential Kubernetes commands.

