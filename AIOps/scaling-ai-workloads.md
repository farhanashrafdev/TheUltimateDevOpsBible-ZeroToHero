# Scaling AI Workloads: Kubernetes for ML

## 🎯 Introduction

AI workloads (training, inference) have unique requirements: GPUs, massive parallelism, and large datasets. Kubernetes is the standard OS for AI.

## 🏗️ Architecture

### 1. GPU Scheduling

**Enable GPUs in K8s**:
```yaml
resources:
  limits:
    nvidia.com/gpu: 1
```

**Multi-Instance GPU (MIG)**:
- Split one A100 into 7 smaller GPUs for inference.

### 2. Distributed Training

**Frameworks**:
- **Kubeflow**: Full ML platform on K8s.
- **PyTorchJob**: Native operator for distributed PyTorch.

**Example PyTorchJob**:
```yaml
apiVersion: "kubeflow.org/v1"
kind: PyTorchJob
metadata:
  name: pytorch-dist-mnist
spec:
  pytorchReplicaSpecs:
    Master:
      replicas: 1
      template:
        spec:
          containers:
          - name: pytorch
            image: pytorch-dist-mnist:latest
            resources:
              limits:
                nvidia.com/gpu: 1
    Worker:
      replicas: 2
      template:
        spec:
          containers:
          - name: pytorch
            image: pytorch-dist-mnist:latest
            resources:
              limits:
                nvidia.com/gpu: 1
```

### 3. Model Serving at Scale

**KServe**:
- Serverless inference on K8s.
- Scale to zero when no traffic.
- Canary rollouts.

```yaml
apiVersion: "serving.kserve.io/v1beta1"
kind: InferenceService
metadata:
  name: sklearn-iris
spec:
  predictor:
    sklearn:
      storageUri: "gs://kfserving-examples/models/sklearn/iris"
```

---

## 🚀 Autoscaling

### 1. HPA (Horizontal Pod Autoscaler)

Scale based on GPU usage? Hard with standard metrics.
**Solution**: Use **KEDA** or custom metrics adapter.

### 2. Cluster Autoscaler / Karpenter

**Karpenter**:
- Provision nodes just-in-time.
- "I need a GPU node now" -> Karpenter spins it up in seconds.

```yaml
# Karpenter Provisioner
apiVersion: karpenter.sh/v1alpha5
kind: Provisioner
metadata:
  name: gpu
spec:
  requirements:
    - key: "karpenter.k8s.aws/instance-category"
      operator: In
      values: ["g4dn", "p3"]
    - key: "karpenter.sh/capacity-type"
      operator: In
      values: ["spot"]
  limits:
    resources:
      gpu: 100
```

---

## 💾 Data Management

**Storage**:
- **PVC/PV**: Standard storage.
- **CSI S3**: Mount S3 buckets as filesystems (slow for training).
- **FSx for Lustre**: High-performance cache for S3 (fast for training).

---

## ✅ Best Practices

- [ ] Use Spot Instances for training (save 70%)
- [ ] Use Karpenter for fast node provisioning
- [ ] Optimize container images (large ML images are slow to pull)
- [ ] Use MIG for inference to maximize GPU utilization
- [ ] Monitor GPU utilization (DCGM Exporter)

---

**Next**: Explore [Platform Engineering](../Platform-Engineering/overview.md).
