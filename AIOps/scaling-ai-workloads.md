# Scaling AI Workloads

## 🎯 Introduction

AI/ML workloads have unique scaling requirements: GPU resources, large models, batch processing, and variable inference loads.

## 📚 GPU Scheduling in Kubernetes

### Node Setup

```yaml
# GPU node label
kubectl label nodes gpu-node-1 accelerator=nvidia-tesla-v100

# Install NVIDIA device plugin
kubectl apply -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v0.14.0/nvidia-device-plugin.yml
```

### GPU Pod Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ml-inference
spec:
  replicas: 3
  template:
    spec:
      containers:
        - name: model
          image: myorg/ml-model:v1
          resources:
            limits:
              nvidia.com/gpu: 1
              memory: "16Gi"
            requests:
              nvidia.com/gpu: 1
              memory: "8Gi"
      nodeSelector:
        accelerator: nvidia-tesla-v100
      tolerations:
        - key: "nvidia.com/gpu"
          operator: "Exists"
          effect: "NoSchedule"
```

## 🔧 Inference Scaling

### Request-Based Autoscaling

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: inference-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ml-inference
  minReplicas: 2
  maxReplicas: 20
  metrics:
    - type: Pods
      pods:
        metric:
          name: inference_requests_per_second
        target:
          type: AverageValue
          averageValue: 100
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 60
```

### KEDA (Kubernetes Event-Driven Autoscaling)

```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: inference-scaler
spec:
  scaleTargetRef:
    name: ml-inference
  minReplicaCount: 0
  maxReplicaCount: 50
  triggers:
    - type: prometheus
      metadata:
        serverAddress: http://prometheus:9090
        metricName: inference_queue_length
        threshold: "100"
        query: sum(inference_queue_length)
```

## 📊 Model Serving Platforms

### Seldon Core

```yaml
apiVersion: machinelearning.seldon.io/v1
kind: SeldonDeployment
metadata:
  name: model
spec:
  predictors:
    - name: default
      replicas: 3
      graph:
        name: classifier
        type: MODEL
      componentSpecs:
        - spec:
            containers:
              - name: classifier
                image: model:v1
                resources:
                  requests:
                    nvidia.com/gpu: 1
```

### KServe

```yaml
apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata:
  name: sklearn-model
spec:
  predictor:
    minReplicas: 1
    maxReplicas: 10
    scaleMetric: concurrency
    scaleTarget: 10
    sklearn:
      storageUri: gs://bucket/model
      resources:
        requests:
          cpu: 1
          memory: 2Gi
```

## ✅ Best Practices

1. **Right-size GPUs**: Use GPU types appropriate for workload
2. **Batch Requests**: Combine multiple inferences for efficiency
3. **Model Optimization**: Quantization, pruning for smaller models
4. **Scale to Zero**: Save costs when no traffic
5. **Multi-Model Serving**: Share resources across models

---

**Next**: Return to [AIOps Overview](./overview.md).

