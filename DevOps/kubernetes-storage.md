# Kubernetes Storage: Complete Guide

## 🎯 Introduction

Kubernetes provides flexible storage options for stateful applications. Understanding storage is crucial for databases, file storage, and persistent data requirements.

### Storage Concepts

- **Volumes**: Storage attached to pods
- **PersistentVolumes (PV)**: Cluster-wide storage resources
- **PersistentVolumeClaims (PVC)**: Storage requests from users
- **StorageClasses**: Dynamic provisioning
- **Volume Snapshots**: Point-in-time backups

## 📦 Volume Types

### emptyDir

**Temporary storage, shared between containers in a pod**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: empty-dir-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: cache
      mountPath: /cache
  - name: sidecar
    image: busybox
    volumeMounts:
    - name: cache
      mountPath: /data
  volumes:
  - name: cache
    emptyDir:
      sizeLimit: 1Gi
```

**Use Cases**:
- Temporary files
- Cache
- Scratch space
- Inter-container communication

### hostPath

**Mount node filesystem into pod**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: node-fs
      mountPath: /host
  volumes:
  - name: node-fs
    hostPath:
      path: /data
      type: DirectoryOrCreate
```

**Types**:
- `DirectoryOrCreate`: Create if doesn't exist
- `Directory`: Must exist
- `FileOrCreate`: Create file if doesn't exist
- `File`: Must exist
- `Socket`: Unix socket
- `CharDevice`: Character device
- `BlockDevice`: Block device

**⚠️ Warning**: hostPath is not portable and can be a security risk

### PersistentVolume (PV)

**Cluster-wide storage resource**

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
    - ReadOnlyMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: slow
  hostPath:
    path: /mnt/data
```

**Access Modes**:
- **ReadWriteOnce (RWO)**: Single node read-write
- **ReadOnlyMany (ROX)**: Multiple nodes read-only
- **ReadWriteMany (RWX)**: Multiple nodes read-write
- **ReadWriteOncePod (RWOP)**: Single pod read-write (K8s 1.22+)

**Reclaim Policies**:
- **Retain**: Manual cleanup
- **Recycle**: Basic scrub (deprecated)
- **Delete**: Automatically delete

### PersistentVolumeClaim (PVC)

**Storage request from user**

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
  storageClassName: fast-ssd
  # Optional: bind to specific PV
  # volumeName: pv-volume
```

**Using PVC in Pod**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pvc-pod
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

### StorageClass

**Dynamic provisioning**

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  fsType: ext4
  iops: "3000"
  encrypted: "true"
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Delete
```

**Volume Binding Modes**:
- **Immediate**: Bind immediately
- **WaitForFirstConsumer**: Bind when pod is created

**Provisioners**:
- `kubernetes.io/aws-ebs`: AWS EBS
- `kubernetes.io/gce-pd`: GCE Persistent Disk
- `kubernetes.io/azure-disk`: Azure Disk
- `kubernetes.io/cinder`: OpenStack Cinder
- `kubernetes.io/vsphere-volume`: vSphere
- Custom provisioners

### AWS EBS StorageClass

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gp3-ssd
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  fsType: ext4
  iops: "3000"
  throughput: "125"
  encrypted: "true"
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
```

### GCE Persistent Disk StorageClass

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard-ssd
provisioner: pd.csi.storage.gke.io
parameters:
  type: pd-ssd
  replication-type: regional-pd
volumeBindingMode: WaitForFirstConsumer
```

## 💾 StatefulSets with Storage

### StatefulSet with PVC

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: database
spec:
  serviceName: "database"
  replicas: 3
  selector:
    matchLabels:
      app: database
  template:
    metadata:
      labels:
        app: database
    spec:
      containers:
      - name: db
        image: postgres:15
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: fast-ssd
      resources:
        requests:
          storage: 20Gi
```

**StatefulSet Characteristics**:
- Stable network identity
- Ordered deployment
- Ordered scaling
- Ordered updates
- Persistent storage per pod

## 📸 Volume Snapshots

### VolumeSnapshotClass

```yaml
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshotClass
metadata:
  name: csi-snapshotter
driver: ebs.csi.aws.com
deletionPolicy: Retain
```

### VolumeSnapshot

```yaml
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: pvc-snapshot
spec:
  volumeSnapshotClassName: csi-snapshotter
  source:
    persistentVolumeClaimName: pv-claim
```

### Restore from Snapshot

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-restored
spec:
  storageClassName: fast-ssd
  dataSource:
    name: pvc-snapshot
    kind: VolumeSnapshot
    apiGroup: snapshot.storage.k8s.io
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

## 🔄 Volume Operations

### Expanding Volumes

```yaml
# Update PVC
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi  # Increased from 10Gi
  storageClassName: fast-ssd
```

**Requirements**:
- StorageClass with `allowVolumeExpansion: true`
- PVC in `Bound` state
- File system expansion (may require pod restart)

### Volume Cloning

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-clone
spec:
  storageClassName: fast-ssd
  dataSource:
    name: pv-claim
    kind: PersistentVolumeClaim
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

## 🗄️ Storage Best Practices

### 1. Use StorageClasses

```yaml
# Always specify storageClassName
# Enables dynamic provisioning
```

### 2. Choose Right Access Mode

```yaml
# ReadWriteOnce: Single pod
# ReadWriteMany: Multiple pods (NFS, EFS)
# ReadOnlyMany: Read-only multiple pods
```

### 3. Set Appropriate Size

```yaml
# Request enough storage
# Consider growth
# Use volume expansion if needed
```

### 4. Backup Strategy

```yaml
# Use volume snapshots
# Regular backups
# Test restore procedures
```

### 5. Monitor Storage

```bash
# Monitor PV/PVC usage
kubectl get pv
kubectl get pvc
df -h  # In pods
```

### 6. Clean Up

```bash
# Delete PVCs when done
# Clean up unused PVs
# Remove old snapshots
```

## 🔍 Storage Troubleshooting

### Common Issues

```bash
# PVC pending
kubectl describe pvc pvc-name
# Check storage class
# Check node capacity

# Mount errors
kubectl describe pod pod-name
# Check volume mounts
# Check permissions

# Volume expansion failed
kubectl describe pvc pvc-name
# Check storage class allows expansion
```

### Debugging Commands

```bash
# List PVs
kubectl get pv

# List PVCs
kubectl get pvc

# Describe PV
kubectl describe pv pv-name

# Describe PVC
kubectl describe pvc pvc-name

# List StorageClasses
kubectl get storageclass

# Check volume in pod
kubectl exec pod-name -- df -h
kubectl exec pod-name -- ls -la /mount/path
```

## ✅ Mastery Checklist

- [ ] Understand volume types
- [ ] Create and use PVs/PVCs
- [ ] Configure StorageClasses
- [ ] Use StatefulSets with storage
- [ ] Create volume snapshots
- [ ] Expand volumes
- [ ] Clone volumes
- [ ] Troubleshoot storage issues
- [ ] Implement backup strategy
- [ ] Monitor storage usage

---

**Next Steps**:
- Learn [Kubernetes Operations](./kubernetes-operations.md)
- Explore [Service Mesh](./service-mesh.md)
- Master [Production Practices](./production-practices.md)

**Remember**: Storage is critical for stateful applications. Choose the right storage class, set appropriate sizes, implement backups, and always monitor storage usage. Test your backup and restore procedures regularly.
