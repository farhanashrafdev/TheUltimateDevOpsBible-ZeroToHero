# Kubernetes Labs

## 🎯 Kubernetes Practical Exercises

### Lab 1: Pod Basics
**Objective**: Create and manage pods

**Tasks**:
1. Create pod YAML
2. Apply pod
3. View pod status
4. Access pod logs
5. Execute commands in pod

**Commands**:
```bash
kubectl apply -f pod.yaml
kubectl get pods
kubectl describe pod nginx-pod
kubectl logs nginx-pod
kubectl exec -it nginx-pod -- sh
```

### Lab 2: Deployments
**Objective**: Manage application deployments

**Tasks**:
1. Create deployment
2. Scale deployment
3. Update deployment
4. Rollback deployment

**Commands**:
```bash
kubectl apply -f deployment.yaml
kubectl scale deployment nginx --replicas=5
kubectl set image deployment/nginx nginx=nginx:1.22
kubectl rollout undo deployment/nginx
```

### Lab 3: Services
**Objective**: Expose applications

**Tasks**:
1. Create ClusterIP service
2. Create NodePort service
3. Create LoadBalancer service
4. Test connectivity

**Service Example**:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

### Lab 4: ConfigMaps and Secrets
**Objective**: Manage configuration

**Tasks**:
1. Create ConfigMap
2. Create Secret
3. Use in pods
4. Update configuration

**Commands**:
```bash
kubectl create configmap app-config --from-file=config.yaml
kubectl create secret generic app-secret --from-literal=key=value
kubectl get configmap app-config -o yaml
```

### Lab 5: Persistent Volumes
**Objective**: Manage storage

**Tasks**:
1. Create PersistentVolume
2. Create PersistentVolumeClaim
3. Mount in pod
4. Test persistence

**PV Example**:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-volume
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data
```

## ✅ Completion Checklist

- [ ] Created all resources
- [ ] Tested functionality
- [ ] Troubleshot issues
- [ ] Documented learnings

---

**Next**: Complete security labs.

