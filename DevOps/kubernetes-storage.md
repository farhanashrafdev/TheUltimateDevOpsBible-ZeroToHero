# Kubernetes Storage

## 🎯 Storage Concepts

### Volume Types
- **emptyDir**: Temporary storage
- **hostPath**: Node filesystem
- **PersistentVolume**: Cluster-wide storage
- **PersistentVolumeClaim**: Storage request

## 📝 Storage Examples

### PersistentVolume
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
  persistentVolumeReclaimPolicy: Retain
  storageClassName: slow
  hostPath:
    path: /data
```

### PersistentVolumeClaim
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: slow
```

### Using PVC in Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: storage
      mountPath: /data
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: pv-claim
```

## 🔧 Storage Classes

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  fsType: ext4
```

## ✅ Best Practices

- Use PersistentVolumes for data
- Choose appropriate storage class
- Understand access modes
- Implement backup strategy
- Monitor storage usage
- Clean up unused volumes

---

**Next**: Learn service mesh.

