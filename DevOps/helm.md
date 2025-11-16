# Helm: Complete Kubernetes Package Manager Guide

## 🎯 Introduction

Helm is the package manager for Kubernetes, simplifying application deployment and management through reusable packages called charts. It enables you to define, install, and upgrade complex Kubernetes applications.

### Why Helm?

**Key Benefits**:
- **Package Management**: Bundle Kubernetes resources into charts
- **Template System**: Parameterize deployments with values
- **Version Control**: Manage application versions
- **Dependency Management**: Handle chart dependencies
- **Rollback**: Easy rollback to previous versions
- **Release Management**: Track deployed releases
- **Large Ecosystem**: 10,000+ charts available

### Helm Concepts

- **Chart**: Package of Kubernetes resources
- **Release**: Installed chart instance
- **Values**: Configuration parameters
- **Templates**: Kubernetes manifests with variables
- **Repository**: Chart storage location

## 📚 Chart Structure

### Standard Chart Layout

```
mychart/
├── Chart.yaml              # Chart metadata
├── values.yaml             # Default values
├── charts/                 # Chart dependencies
├── templates/              # Template files
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── _helpers.tpl        # Helper templates
│   └── NOTES.txt           # Installation notes
└── templates/tests/        # Test files
    └── test-connection.yaml
```

## 📝 Complete Chart Examples

### Chart.yaml

```yaml
apiVersion: v2
name: myapp
description: A Helm chart for my application
type: application
version: 1.0.0
appVersion: "1.0.0"
keywords:
  - web
  - application
home: https://example.com
sources:
  - https://github.com/user/repo
maintainers:
  - name: DevOps Team
    email: devops@example.com
dependencies:
  - name: postgresql
    version: "12.0.0"
    repository: "https://charts.bitnami.com/bitnami"
    condition: postgresql.enabled
```

### values.yaml

```yaml
# Default values
replicaCount: 3

image:
  repository: nginx
  pullPolicy: IfNotPresent
  tag: "1.21"

imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""

serviceAccount:
  create: true
  annotations: {}
  name: ""

podAnnotations: {}

podSecurityContext: {}
  # fsGroup: 2000

securityContext: {}
  # capabilities:
  #   drop:
  #   - ALL
  # readOnlyRootFilesystem: true
  # runAsNonRoot: true
  # runAsUser: 1000

service:
  type: ClusterIP
  port: 80
  targetPort: 80

ingress:
  enabled: false
  className: "nginx"
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: myapp.example.com
      paths:
        - path: /
          pathType: Prefix
  tls: []
  #  - secretName: myapp-tls
  #    hosts:
  #      - myapp.example.com

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi

autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80

nodeSelector: {}

tolerations: []

affinity: {}

# PostgreSQL dependency
postgresql:
  enabled: true
  auth:
    postgresPassword: "changeme"
    database: "myapp"
```

### templates/deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "myapp.fullname" . }}
  labels:
    {{- include "myapp.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "myapp.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      annotations:
        checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
        {{- with .Values.podAnnotations }}
        {{- toYaml . | nindent 8 }}
        {{- end }}
      labels:
        {{- include "myapp.selectorLabels" . | nindent 8 }}
    spec:
      {{- with .Values.imagePullSecrets }}
      imagePullSecrets:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      serviceAccountName: {{ include "myapp.serviceAccountName" . }}
      securityContext:
        {{- toYaml .Values.podSecurityContext | nindent 8 }}
      containers:
        - name: {{ .Chart.Name }}
          securityContext:
            {{- toYaml .Values.securityContext | nindent 12 }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: {{ .Values.service.targetPort }}
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /
              port: http
          readinessProbe:
            httpGet:
              path: /
              port: http
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: {{ include "myapp.fullname" . }}-postgresql
                  key: postgresql-password
      {{- with .Values.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.affinity }}
      affinity:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.tolerations }}
      tolerations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
```

### templates/service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "myapp.fullname" . }}
  labels:
    {{- include "myapp.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
      protocol: TCP
      name: http
  selector:
    {{- include "myapp.selectorLabels" . | nindent 4 }}
```

### templates/_helpers.tpl

```yaml
{{/*
Expand the name of the chart.
*/}}
{{- define "myapp.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Create a default fully qualified app name.
*/}}
{{- define "myapp.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- if contains $name .Release.Name }}
{{- .Release.Name | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}
{{- end }}

{{/*
Create chart name and version as used by the chart label.
*/}}
{{- define "myapp.chart" -}}
{{- printf "%s-%s" .Chart.Name .Chart.Version | replace "+" "_" | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Common labels
*/}}
{{- define "myapp.labels" -}}
helm.sh/chart: {{ include "myapp.chart" . }}
{{ include "myapp.selectorLabels" . }}
{{- if .Chart.AppVersion }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
{{- end }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}

{{/*
Selector labels
*/}}
{{- define "myapp.selectorLabels" -}}
app.kubernetes.io/name: {{ include "myapp.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}

{{/*
Create the name of the service account to use
*/}}
{{- define "myapp.serviceAccountName" -}}
{{- if .Values.serviceAccount.create }}
{{- default (include "myapp.fullname" .) .Values.serviceAccount.name }}
{{- else }}
{{- default "default" .Values.serviceAccount.name }}
{{- end }}
{{- end }}
```

## 🔧 Helm Commands

### Chart Management

```bash
# Create new chart
helm create mychart

# Package chart
helm package mychart

# Install chart
helm install myapp ./mychart
helm install myapp ./mychart -n production
helm install myapp ./mychart --set replicaCount=5
helm install myapp ./mychart -f values-production.yaml

# Upgrade release
helm upgrade myapp ./mychart
helm upgrade myapp ./mychart --set image.tag=v2.0.0
helm upgrade myapp ./mychart --reuse-values

# Rollback
helm rollback myapp
helm rollback myapp 2  # To revision 2

# Uninstall
helm uninstall myapp
helm uninstall myapp -n production
```

### Release Management

```bash
# List releases
helm list
helm list -A  # All namespaces
helm list --all  # Including failed

# Get release info
helm get all myapp
helm get values myapp
helm get manifest myapp
helm get notes myapp
helm get hooks myapp

# History
helm history myapp
```

### Repository Management

```bash
# Add repository
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add stable https://charts.helm.sh/stable

# List repositories
helm repo list

# Update repositories
helm repo update

# Search charts
helm search repo nginx
helm search repo bitnami/nginx

# Remove repository
helm repo remove bitnami
```

### Template and Testing

```bash
# Template (dry-run)
helm template myapp ./mychart
helm template myapp ./mychart --values values.yaml
helm template myapp ./mychart --debug

# Lint chart
helm lint ./mychart

# Test chart
helm test myapp
```

### Dependency Management

```bash
# Update dependencies
helm dependency update ./mychart

# Build dependencies
helm dependency build ./mychart

# List dependencies
helm dependency list ./mychart
```

## 🎯 Advanced Features

### Chart Dependencies

**Chart.yaml**:
```yaml
dependencies:
  - name: postgresql
    version: "12.0.0"
    repository: "https://charts.bitnami.com/bitnami"
    condition: postgresql.enabled
    tags:
      - database
```

**values.yaml**:
```yaml
postgresql:
  enabled: true
  auth:
    postgresPassword: "changeme"
```

### Hooks

```yaml
# Pre-install hook
apiVersion: batch/v1
kind: Job
metadata:
  name: pre-install-job
  annotations:
    "helm.sh/hook": pre-install
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": before-hook-creation
spec:
  template:
    spec:
      containers:
        - name: pre-install
          image: busybox
          command: ['sh', '-c', 'echo Pre-install hook']
      restartPolicy: Never
```

### Tests

**templates/tests/test-connection.yaml**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "myapp.fullname" . }}-test-connection"
  labels:
    {{- include "myapp.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": test
spec:
  containers:
    - name: wget
      image: busybox
      command: ['wget']
      args: ['{{ include "myapp.fullname" . }}:{{ .Values.service.port }}']
  restartPolicy: Never
```

## ✅ Best Practices

### 1. Version Charts Properly

```yaml
# Use semantic versioning
version: 1.0.0
appVersion: "1.0.0"
```

### 2. Use Values for Configuration

```yaml
# Don't hardcode values
# Use .Values for all configuration
replicas: {{ .Values.replicaCount }}
```

### 3. Document Charts

```yaml
# Chart.yaml
description: Clear description
home: Project homepage
sources: Source code location
maintainers: Maintainer information
```

### 4. Test Before Release

```bash
helm lint ./mychart
helm template myapp ./mychart --debug
helm install myapp ./mychart --dry-run --debug
```

### 5. Use Helpers

```yaml
# _helpers.tpl for reusable templates
{{- define "myapp.labels" -}}
# ...
{{- end }}
```

## ✅ Mastery Checklist

- [ ] Create Helm charts
- [ ] Use templates and values
- [ ] Manage chart dependencies
- [ ] Implement hooks
- [ ] Create chart tests
- [ ] Publish charts to repository
- [ ] Use Helm with CI/CD
- [ ] Manage releases
- [ ] Rollback releases
- [ ] Optimize chart structure

---

**Next Steps**:
- Learn [Kubernetes Operations](./kubernetes-operations.md)
- Explore [Argo CD](./argo-cd.md) for GitOps
- Master [Terraform](./terraform-basics.md) for infrastructure

**Remember**: Helm simplifies Kubernetes deployments. Use values for configuration, test charts thoroughly, and version properly. Start with simple charts and gradually add complexity.
