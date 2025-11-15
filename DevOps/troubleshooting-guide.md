# Troubleshooting Guide: Complete Debugging Reference

## 🎯 Introduction

Troubleshooting is a critical skill for DevOps engineers. This comprehensive guide provides systematic approaches, tools, and techniques for debugging issues in production systems.

### Troubleshooting Methodology

```
1. Gather Information
   ├── Check logs
   ├── Review metrics
   ├── Check events
   └── Understand symptoms

2. Form Hypothesis
   ├── What could cause this?
   ├── What changed recently?
   └── What's the pattern?

3. Test Hypothesis
   ├── Reproduce issue
   ├── Test fixes
   └── Verify solution

4. Resolve
   ├── Apply fix
   ├── Verify resolution
   └── Document learnings
```

## ☸️ Kubernetes Troubleshooting

### Pod Issues

#### Pod Not Starting

```bash
# Step 1: Check pod status
kubectl get pods
kubectl get pods -o wide

# Step 2: Describe pod
kubectl describe pod pod-name
kubectl describe pod pod-name -n namespace

# Step 3: Check events
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl get events --field-selector involvedObject.name=pod-name

# Step 4: Check logs
kubectl logs pod-name
kubectl logs pod-name --previous
kubectl logs pod-name -c container-name

# Step 5: Check resource availability
kubectl top nodes
kubectl describe node node-name
```

**Common Causes**:
- Image pull errors
- Resource constraints
- Configuration errors
- Node issues
- Network policies

#### Pod CrashLoopBackOff

```bash
# Check previous logs
kubectl logs pod-name --previous

# Check restart count
kubectl get pod pod-name -o jsonpath='{.status.containerStatuses[*].restartCount}'

# Describe for events
kubectl describe pod pod-name | grep -A 20 Events

# Check resource limits
kubectl describe pod pod-name | grep -A 10 Limits
```

**Common Causes**:
- Application crashes
- Out of memory
- Configuration errors
- Health check failures

#### Pod Pending

```bash
# Check why pending
kubectl describe pod pod-name | grep -A 10 Events

# Check node resources
kubectl describe node node-name

# Check resource quotas
kubectl describe quota -n namespace

# Check persistent volume claims
kubectl get pvc
kubectl describe pvc pvc-name
```

**Common Causes**:
- Insufficient resources
- Node selector mismatch
- Resource quotas
- PVC not bound

### Service Issues

#### Service Not Accessible

```bash
# Check service
kubectl get svc service-name
kubectl describe svc service-name

# Check endpoints
kubectl get endpoints service-name
kubectl describe endpoints service-name

# Check pods
kubectl get pods -l app=label-value
kubectl get pods -l app=label-value -o wide

# Test connectivity
kubectl run -it --rm debug --image=busybox --restart=Never -- wget -O- http://service-name:80
```

**Common Causes**:
- No pods matching selector
- Port mismatch
- Network policies blocking
- Service type incorrect

### Network Issues

#### Pod Can't Reach Service

```bash
# Test DNS
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup service-name

# Test connectivity
kubectl run -it --rm debug --image=busybox --restart=Never -- wget -O- http://service-name:80

# Check network policies
kubectl get networkpolicies
kubectl describe networkpolicy policy-name

# Check service endpoints
kubectl get endpoints service-name
```

#### DNS Issues

```bash
# Check CoreDNS
kubectl get pods -n kube-system -l k8s-app=kube-dns

# Check CoreDNS logs
kubectl logs -n kube-system -l k8s-app=kube-dns

# Test DNS resolution
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup kubernetes.default
```

### Resource Issues

#### High CPU Usage

```bash
# Check CPU usage
kubectl top pods
kubectl top pods --sort-by=cpu
kubectl top nodes

# Check limits
kubectl describe pod pod-name | grep -A 5 Limits

# Check processes in pod
kubectl exec pod-name -- top
kubectl exec pod-name -- ps aux
```

**Solutions**:
- Scale up deployment
- Increase CPU limits
- Optimize application
- Add more nodes

#### High Memory Usage

```bash
# Check memory usage
kubectl top pods --sort-by=memory
kubectl top nodes

# Check limits
kubectl describe pod pod-name | grep -A 5 Limits

# Check memory in pod
kubectl exec pod-name -- free -h
```

**Solutions**:
- Scale up deployment
- Increase memory limits
- Fix memory leaks
- Add more nodes

## 🐳 Docker Troubleshooting

### Container Issues

#### Container Won't Start

```bash
# View logs
docker logs container-name
docker logs container-name --tail 100
docker logs container-name --since 1h

# Inspect container
docker inspect container-name
docker inspect container-name --format='{{.State.Status}}'

# Check events
docker events
docker events --filter container=container-name
```

#### Container Crashes

```bash
# Check exit code
docker inspect container-name --format='{{.State.ExitCode}}'

# View logs
docker logs container-name

# Check resource limits
docker inspect container-name --format='{{.HostConfig.Memory}}'
docker inspect container-name --format='{{.HostConfig.CpuShares}}'
```

### Image Issues

#### Image Pull Fails

```bash
# Test image pull
docker pull image:tag

# Check registry access
docker login registry.example.com

# Check image exists
docker images | grep image-name
```

#### Build Fails

```bash
# Build with verbose output
docker build --progress=plain -t image:tag .

# Build without cache
docker build --no-cache -t image:tag .

# Check Dockerfile syntax
docker build --dry-run -t image:tag .
```

## 🔄 CI/CD Troubleshooting

### Pipeline Issues

#### Pipeline Fails

```bash
# Check logs
# GitHub Actions: View workflow run
# GitLab CI: View pipeline logs
# Jenkins: View build console

# Enable debug mode
# GitHub Actions: ACTIONS_STEP_DEBUG=true
# GitLab CI: CI_DEBUG_TRACE=true
```

#### Build Timeout

```bash
# Increase timeout
# GitHub Actions: timeout-minutes
# GitLab CI: timeout
# Jenkins: timeout step
```

#### Deployment Fails

```bash
# Check deployment status
kubectl rollout status deployment/deployment-name

# View deployment history
kubectl rollout history deployment/deployment-name

# Check events
kubectl get events --sort-by=.metadata.creationTimestamp
```

## ☁️ Cloud Issues

### AWS Issues

```bash
# Check credentials
aws sts get-caller-identity

# Check region
aws configure get region

# View logs
aws logs tail /aws/lambda/function-name --follow

# Check instance status
aws ec2 describe-instance-status --instance-ids i-xxx

# Check service health
aws health describe-events
```

### Network Connectivity

```bash
# Test connectivity
ping host
traceroute host
nc -zv host port
telnet host port

# Check DNS
nslookup host
dig host
host host

# Check firewall
iptables -L
ufw status
```

## 📊 Performance Issues

### High CPU

```bash
# Find high CPU processes
top
htop
ps aux --sort=-%cpu | head

# Kubernetes
kubectl top pods --sort-by=cpu
kubectl top nodes

# Check load
uptime
cat /proc/loadavg
```

### High Memory

```bash
# Check memory
free -h
cat /proc/meminfo

# Find high memory processes
ps aux --sort=-%mem | head

# Kubernetes
kubectl top pods --sort-by=memory
```

### Slow Performance

```bash
# Check I/O
iostat 1
iotop

# Check network
iftop
nethogs

# Check database
# Slow query logs
# Connection pool
```

## 🔍 Debugging Tools

### Kubernetes Debugging

```bash
# Debug pod
kubectl debug pod/pod-name -it --image=busybox
kubectl debug pod/pod-name -it --image=busybox --target=container-name

# Port forward
kubectl port-forward pod/pod-name 8080:80

# Copy files
kubectl cp pod-name:/path/to/file ./local-file
kubectl cp ./local-file pod-name:/path/to/file

# Exec into pod
kubectl exec -it pod-name -- bash
kubectl exec -it pod-name -c container-name -- sh
```

### Network Debugging

```bash
# Packet capture
tcpdump -i eth0
tcpdump -i eth0 port 80
tcpdump -i eth0 -w capture.pcap

# Network analysis
wireshark
tshark

# Connection testing
nc -zv host port
telnet host port
curl -v http://host:port
```

### System Debugging

```bash
# Process analysis
strace -p PID
ltrace -p PID
perf top

# Memory analysis
valgrind
memcheck

# System calls
strace command
```

## 🚨 Common Scenarios

### Scenario 1: Application Not Responding

```bash
# 1. Check if pod is running
kubectl get pods -l app=myapp

# 2. Check pod logs
kubectl logs -l app=myapp --tail=100

# 3. Check service
kubectl get svc myapp
kubectl describe svc myapp

# 4. Check endpoints
kubectl get endpoints myapp

# 5. Test connectivity
kubectl run -it --rm test --image=busybox --restart=Never -- wget -O- http://myapp:80

# 6. Check health probes
kubectl describe pod pod-name | grep -A 10 Liveness
```

### Scenario 2: High Error Rate

```bash
# 1. Check error logs
kubectl logs -l app=myapp | grep -i error

# 2. Check metrics
# Grafana dashboard
# Prometheus queries

# 3. Check recent changes
kubectl rollout history deployment/myapp

# 4. Check resource usage
kubectl top pods -l app=myapp

# 5. Check dependencies
kubectl get pods -l app=database
```

### Scenario 3: Slow Response Times

```bash
# 1. Check latency metrics
# Prometheus: histogram_quantile(0.95, ...)

# 2. Check database
kubectl exec -it db-pod -- psql -c "SELECT * FROM pg_stat_activity"

# 3. Check network
kubectl exec -it app-pod -- ping database

# 4. Check resource limits
kubectl describe pod pod-name | grep Limits

# 5. Profile application
# Add profiling endpoints
```

## 📝 Debugging Checklist

### Systematic Approach

```markdown
## Troubleshooting Checklist

### Information Gathering
- [ ] Check logs (application, system, container)
- [ ] Review metrics and dashboards
- [ ] Check recent changes
- [ ] Review events and alerts
- [ ] Understand symptoms

### Hypothesis Formation
- [ ] What could cause this?
- [ ] What changed recently?
- [ ] What's the pattern?
- [ ] Is it affecting all or some?

### Testing
- [ ] Can I reproduce?
- [ ] Test individual components
- [ ] Test fixes in isolation
- [ ] Verify solution

### Resolution
- [ ] Apply fix
- [ ] Verify resolution
- [ ] Monitor for stability
- [ ] Document learnings
```

## ✅ Best Practices

### 1. Start with Logs

```bash
# Always check logs first
# Application logs
# System logs
# Container logs
```

### 2. Use Systematic Approach

```yaml
# Don't guess
# Follow methodology
# Document findings
```

### 3. Document Findings

```markdown
# Document:
# - Symptoms
# - Investigation steps
# - Root cause
# - Resolution
# - Prevention
```

### 4. Learn from Incidents

```yaml
# Post-mortems
# Update runbooks
# Improve monitoring
# Prevent recurrence
```

### 5. Build Runbooks

```markdown
# Create runbooks for:
# - Common issues
# - Known problems
# - Standard procedures
```

## ✅ Mastery Checklist

- [ ] Systematic troubleshooting approach
- [ ] Kubernetes debugging skills
- [ ] Docker troubleshooting
- [ ] Network debugging
- [ ] Performance analysis
- [ ] Log analysis
- [ ] Metrics interpretation
- [ ] Create runbooks
- [ ] Document incidents
- [ ] Learn from failures

---

**Next Steps**:
- Learn [Production Practices](./production-practices.md)
- Explore [On-Call & SRE](./oncall-sre.md)
- Master [Monitoring & Observability](./monitoring-observability.md)

**Remember**: Troubleshooting is a skill that improves with practice. Use systematic approaches, document everything, and always learn from incidents. Good troubleshooting skills enable fast incident resolution and continuous improvement.
