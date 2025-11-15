# Helm

## 🎯 Introduction

Helm is the package manager for Kubernetes, simplifying application deployment and management through charts.

## 📚 Core Concepts

### Helm Components
- **Charts**: Package of Kubernetes resources
- **Releases**: Installed chart instances
- **Values**: Configuration parameters
- **Templates**: Kubernetes manifests with variables

## 📝 Basic Chart Structure

```
mychart/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── _helpers.tpl
└── charts/
```

## 🔧 Chart Example

### Chart.yaml
```yaml
apiVersion: v2
name: myapp
version: 1.0.0
description: My application chart
```

### values.yaml
```yaml
replicaCount: 3
image:
  repository: nginx
  tag: "1.21"
service:
  type: ClusterIP
  port: 80
```

### templates/deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.name }}
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

## 🔧 Helm Commands

```bash
# Install chart
helm install myapp ./mychart

# Upgrade release
helm upgrade myapp ./mychart

# List releases
helm list

# Uninstall
helm uninstall myapp

# Template (dry-run)
helm template myapp ./mychart
```

## ✅ Best Practices

- Version charts properly
- Use values.yaml for configuration
- Document charts
- Test with helm template
- Use chart dependencies
- Follow naming conventions

---

**Next**: Learn Terraform for Infrastructure as Code.

