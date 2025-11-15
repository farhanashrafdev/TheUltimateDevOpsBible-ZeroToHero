# Troubleshooting Cheat Sheet

## 📚 Common Issues

### Pod Not Starting
```bash
kubectl describe pod pod-name
kubectl logs pod-name
kubectl get events
```

### High CPU/Memory
```bash
kubectl top pods
kubectl top nodes
kubectl describe pod pod-name | grep Limits
```

### Network Issues
```bash
kubectl exec pod -- ping service
kubectl get svc
kubectl describe svc service-name
```

### Docker Issues
```bash
docker ps -a
docker logs container
docker inspect container
docker stats
```

---

**Quick Reference**: Common troubleshooting commands.

