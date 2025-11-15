# Kubernetes Cheat Sheet - Complete Reference

## 📚 kubectl Basics

### Configuration

```bash
# View current context
kubectl config current-context

# List all contexts
kubectl config get-contexts

# Switch context
kubectl config use-context my-context

# Set default namespace
kubectl config set-context --current --namespace=production

# View configuration
kubectl config view
kubectl config view --minify

# Get cluster info
kubectl cluster-info
kubectl cluster-info dump
```

### Getting Resources

```bash
# Get all resources
kubectl get all
kubectl get all -A                    # All namespaces
kubectl get all -n production          # Specific namespace

# Get specific resources
kubectl get pods
kubectl get pods -A                    # All namespaces
kubectl get pods -n production         # Specific namespace
kubectl get pods -o wide               # More details
kubectl get pods -o yaml               # YAML output
kubectl get pods -o json               # JSON output
kubectl get pods --watch               # Watch changes
kubectl get pods -l app=nginx          # Filter by label
kubectl get pods --field-selector status.phase=Running

# Get deployments
kubectl get deployments
kubectl get deployments -n production
kubectl get deployments -o wide

# Get services
kubectl get services
kubectl get svc                        # Short form
kubectl get svc -n production

# Get other resources
kubectl get nodes
kubectl get namespaces
kubectl get configmaps
kubectl get secrets
kubectl get ingress
kubectl get pv                         # Persistent volumes
kubectl get pvc                        # Persistent volume claims
kubectl get jobs
kubectl get cronjobs
kubectl get daemonsets
kubectl get statefulsets
```

### Describing Resources

```bash
# Describe resources
kubectl describe pod pod-name
kubectl describe pod pod-name -n production
kubectl describe deployment nginx-deployment
kubectl describe service nginx-service
kubectl describe node node-name
kubectl describe namespace production

# Get resource details
kubectl get pod pod-name -o yaml
kubectl get pod pod-name -o json
kubectl get pod pod-name -o jsonpath='{.status.phase}'
kubectl get pod pod-name -o jsonpath='{.spec.containers[*].image}'
```

## 🐳 Pods

### Creating Pods

```bash
# Create pod from YAML
kubectl create -f pod.yaml
kubectl apply -f pod.yaml

# Create pod imperatively
kubectl run nginx-pod --image=nginx:1.21
kubectl run nginx-pod --image=nginx:1.21 --port=80
kubectl run nginx-pod --image=nginx:1.21 --env="KEY=value"
kubectl run nginx-pod --image=nginx:1.21 --command -- sleep 3600

# Create pod with resource limits
kubectl run nginx-pod --image=nginx:1.21 \
  --requests="memory=64Mi,cpu=250m" \
  --limits="memory=128Mi,cpu=500m"
```

### Managing Pods

```bash
# List pods
kubectl get pods
kubectl get pods -l app=nginx
kubectl get pods --sort-by=.metadata.creationTimestamp
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase

# View pod details
kubectl describe pod pod-name
kubectl get pod pod-name -o yaml
kubectl get pod pod-name -o json

# Delete pod
kubectl delete pod pod-name
kubectl delete pod pod-name -n production
kubectl delete pods -l app=nginx
kubectl delete pods --all
kubectl delete pods --all -n production

# Edit pod
kubectl edit pod pod-name

# Replace pod
kubectl replace -f pod.yaml
```

### Pod Logs

```bash
# View logs
kubectl logs pod-name
kubectl logs pod-name -n production
kubectl logs pod-name -c container-name  # Multi-container pod
kubectl logs pod-name --previous          # Previous instance
kubectl logs pod-name -f                 # Follow logs
kubectl logs pod-name --tail=100         # Last 100 lines
kubectl logs pod-name --since=1h         # Last hour
kubectl logs pod-name --since=2024-01-01T00:00:00Z

# Logs from deployment
kubectl logs deployment/nginx-deployment
kubectl logs deployment/nginx-deployment -f
kubectl logs -l app=nginx                # All pods with label
```

### Executing Commands

```bash
# Execute command in pod
kubectl exec pod-name -- ls /var/log
kubectl exec pod-name -- sh -c "echo 'Hello'"
kubectl exec -it pod-name -- bash
kubectl exec -it pod-name -- sh

# Execute in specific container
kubectl exec -it pod-name -c container-name -- bash

# Run command and get output
kubectl exec pod-name -- env
kubectl exec pod-name -- ps aux
kubectl exec pod-name -- cat /etc/hosts
```

### Port Forwarding

```bash
# Port forward to pod
kubectl port-forward pod/pod-name 8080:80
kubectl port-forward pod/pod-name 8080:80 -n production

# Port forward to service
kubectl port-forward svc/service-name 8080:80
kubectl port-forward svc/service-name 8080:80 -n production

# Port forward to deployment
kubectl port-forward deployment/nginx-deployment 8080:80

# Port forward in background
kubectl port-forward pod/pod-name 8080:80 &
```

### Copying Files

```bash
# Copy from pod
kubectl cp pod-name:/path/to/file ./local-file
kubectl cp pod-name:/var/log/app.log ./app.log -n production

# Copy to pod
kubectl cp ./local-file pod-name:/path/to/file
kubectl cp ./config.yaml pod-name:/etc/config.yaml

# Copy from container in multi-container pod
kubectl cp pod-name:/path/to/file ./local-file -c container-name
```

## 🚀 Deployments

### Creating Deployments

```bash
# Create deployment
kubectl create deployment nginx --image=nginx:1.21
kubectl create deployment nginx --image=nginx:1.21 --replicas=3
kubectl create deployment nginx --image=nginx:1.21 --port=80

# Create from YAML
kubectl create -f deployment.yaml
kubectl apply -f deployment.yaml
```

### Managing Deployments

```bash
# List deployments
kubectl get deployments
kubectl get deployments -n production
kubectl get deployments -o wide

# Scale deployment
kubectl scale deployment nginx-deployment --replicas=5
kubectl scale deployment nginx-deployment --replicas=3 -n production

# Update deployment
kubectl set image deployment/nginx-deployment nginx=nginx:1.22
kubectl set image deployment/nginx-deployment nginx=nginx:1.22 -n production
kubectl set env deployment/nginx-deployment ENV=production
kubectl set resources deployment/nginx-deployment --limits=memory=512Mi,cpu=500m

# Rollout management
kubectl rollout status deployment/nginx-deployment
kubectl rollout status deployment/nginx-deployment -n production
kubectl rollout history deployment/nginx-deployment
kubectl rollout history deployment/nginx-deployment --revision=2
kubectl rollout undo deployment/nginx-deployment
kubectl rollout undo deployment/nginx-deployment --to-revision=2
kubectl rollout pause deployment/nginx-deployment
kubectl rollout resume deployment/nginx-deployment
kubectl rollout restart deployment/nginx-deployment

# Delete deployment
kubectl delete deployment nginx-deployment
kubectl delete deployment nginx-deployment -n production
```

## 🌐 Services

### Creating Services

```bash
# Expose deployment as service
kubectl expose deployment nginx-deployment --port=80 --type=ClusterIP
kubectl expose deployment nginx-deployment --port=80 --target-port=8080 --type=NodePort
kubectl expose deployment nginx-deployment --port=80 --type=LoadBalancer

# Create from YAML
kubectl create -f service.yaml
kubectl apply -f service.yaml
```

### Managing Services

```bash
# List services
kubectl get services
kubectl get svc
kubectl get svc -n production
kubectl get svc -o wide

# Describe service
kubectl describe svc service-name
kubectl describe svc service-name -n production

# Get service endpoints
kubectl get endpoints service-name
kubectl get endpoints service-name -n production

# Edit service
kubectl edit svc service-name

# Delete service
kubectl delete svc service-name
kubectl delete svc service-name -n production
```

## 📦 ConfigMaps and Secrets

### ConfigMaps

```bash
# Create ConfigMap from literal
kubectl create configmap app-config --from-literal=key1=value1 --from-literal=key2=value2

# Create ConfigMap from file
kubectl create configmap app-config --from-file=config.properties
kubectl create configmap app-config --from-file=key1=file1.txt --from-file=key2=file2.txt

# Create ConfigMap from directory
kubectl create configmap app-config --from-file=./config-dir/

# Create ConfigMap from YAML
kubectl create -f configmap.yaml
kubectl apply -f configmap.yaml

# List ConfigMaps
kubectl get configmaps
kubectl get cm

# Describe ConfigMap
kubectl describe cm app-config

# View ConfigMap data
kubectl get cm app-config -o yaml
kubectl get cm app-config -o json

# Delete ConfigMap
kubectl delete cm app-config
```

### Secrets

```bash
# Create secret from literal
kubectl create secret generic db-secret --from-literal=username=admin --from-literal=password=secret123

# Create secret from file
kubectl create secret generic db-secret --from-file=username=./username.txt --from-file=password=./password.txt

# Create TLS secret
kubectl create secret tls tls-secret --cert=cert.pem --key=key.pem

# Create docker-registry secret
kubectl create secret docker-registry regcred \
  --docker-server=docker.io \
  --docker-username=user \
  --docker-password=password \
  --docker-email=user@example.com

# List secrets
kubectl get secrets
kubectl get secrets -n production

# Describe secret
kubectl describe secret db-secret

# View secret (base64 encoded)
kubectl get secret db-secret -o yaml
kubectl get secret db-secret -o json

# Decode secret
kubectl get secret db-secret -o jsonpath='{.data.password}' | base64 -d

# Delete secret
kubectl delete secret db-secret
```

## 🏷️ Namespaces

```bash
# List namespaces
kubectl get namespaces
kubectl get ns

# Create namespace
kubectl create namespace production
kubectl create ns production

# Describe namespace
kubectl describe ns production

# Delete namespace
kubectl delete ns production

# Set default namespace
kubectl config set-context --current --namespace=production

# Get resources in namespace
kubectl get all -n production
```

## 📊 Resource Monitoring

```bash
# Top resources
kubectl top nodes
kubectl top pods
kubectl top pods -n production
kubectl top pods --containers
kubectl top pods --sort-by=memory
kubectl top pods --sort-by=cpu

# Events
kubectl get events
kubectl get events -n production
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl get events --field-selector involvedObject.kind=Pod
```

## 🔍 Debugging

```bash
# Debug pod issues
kubectl describe pod pod-name
kubectl logs pod-name
kubectl logs pod-name --previous
kubectl get events --field-selector involvedObject.name=pod-name

# Debug service issues
kubectl describe svc service-name
kubectl get endpoints service-name
kubectl get pods -l app=nginx

# Debug network issues
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup service-name
kubectl run -it --rm debug --image=busybox --restart=Never -- wget -O- http://service-name:80

# Debug with ephemeral container
kubectl debug pod/pod-name -it --image=busybox
kubectl debug pod/pod-name -it --image=busybox --target=container-name
```

## 🎯 Labels and Selectors

```bash
# Add label
kubectl label pod pod-name environment=production
kubectl label pod pod-name tier=frontend --overwrite

# Remove label
kubectl label pod pod-name environment-

# Get by label
kubectl get pods -l app=nginx
kubectl get pods -l 'app=nginx,environment=production'
kubectl get pods -l 'app in (nginx,apache)'
kubectl get pods -l 'environment!=production'

# Selector expressions
kubectl get pods --selector="app=nginx,environment=production"
```

## 🔧 Advanced Operations

### Dry Run

```bash
# Dry run create
kubectl run nginx --image=nginx:1.21 --dry-run=client -o yaml

# Dry run apply
kubectl apply -f deployment.yaml --dry-run=client
kubectl apply -f deployment.yaml --dry-run=server

# Diff
kubectl diff -f deployment.yaml
```

### Output Formats

```bash
# YAML
kubectl get pod pod-name -o yaml

# JSON
kubectl get pod pod-name -o json

# JSONPath
kubectl get pods -o jsonpath='{.items[*].metadata.name}'
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.phase}{"\n"}{end}'

# Custom columns
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase,AGE:.metadata.creationTimestamp

# Wide
kubectl get pods -o wide

# Name only
kubectl get pods -o name
```

### Batch Operations

```bash
# Delete multiple resources
kubectl delete pods pod1 pod2 pod3
kubectl delete pods -l app=nginx

# Apply to multiple files
kubectl apply -f dir/
kubectl apply -f file1.yaml -f file2.yaml

# Get from multiple namespaces
kubectl get pods --all-namespaces
kubectl get pods -A
```

## 📝 YAML Shortcuts

### Quick Pod

```yaml
kubectl run nginx --image=nginx:1.21 --dry-run=client -o yaml > pod.yaml
```

### Quick Deployment

```yaml
kubectl create deployment nginx --image=nginx:1.21 --dry-run=client -o yaml > deployment.yaml
```

### Quick Service

```yaml
kubectl create service clusterip nginx --tcp=80:8080 --dry-run=client -o yaml > service.yaml
```

## 🚨 Troubleshooting Commands

```bash
# Pod not starting
kubectl describe pod pod-name
kubectl logs pod-name
kubectl get events --sort-by=.metadata.creationTimestamp

# Service not accessible
kubectl get svc
kubectl get endpoints
kubectl describe svc service-name

# Resource issues
kubectl top pods
kubectl describe node node-name
kubectl get events

# Network issues
kubectl exec -it pod-name -- nslookup service-name
kubectl exec -it pod-name -- ping service-name
kubectl get networkpolicies
```

## 📚 Quick Reference

### Common Resource Shortcuts

```bash
po   = pods
deploy = deployments
svc  = services
ns   = namespaces
cm   = configmaps
sec  = secrets
pv   = persistentvolumes
pvc  = persistentvolumeclaims
ing  = ingress
job  = jobs
cj   = cronjobs
ds   = daemonsets
sts  = statefulsets
```

### Common Flags

```bash
-A, --all-namespaces    # All namespaces
-n, --namespace         # Specific namespace
-l, --selector          # Label selector
-o, --output            # Output format
-f, --filename          # File input
--dry-run               # Dry run
--force                 # Force operation
--grace-period          # Grace period for deletion
```

---

**Remember**: This cheat sheet covers the most common kubectl commands. For complete documentation, see: https://kubernetes.io/docs/reference/kubectl/
