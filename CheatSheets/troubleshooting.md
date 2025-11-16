# Troubleshooting Cheat Sheet - Complete Reference

## ☸️ Kubernetes Issues

### Pod Not Starting

```bash
# Describe pod
kubectl describe pod pod-name
kubectl describe pod pod-name -n namespace

# View logs
kubectl logs pod-name
kubectl logs pod-name --previous
kubectl logs pod-name -c container-name

# Check events
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl get events -n namespace

# Check pod status
kubectl get pod pod-name -o yaml
kubectl get pod pod-name -o jsonpath='{.status}'

# Common issues
kubectl describe pod pod-name | grep -A 10 Events
kubectl get pod pod-name -o jsonpath='{.status.containerStatuses[*].state}'
```

### Pod CrashLoopBackOff

```bash
# View previous logs
kubectl logs pod-name --previous

# Check restart count
kubectl get pod pod-name -o jsonpath='{.status.containerStatuses[*].restartCount}'

# Describe for events
kubectl describe pod pod-name

# Check resource limits
kubectl describe pod pod-name | grep -A 5 Limits
kubectl top pod pod-name
```

### Image Pull Errors

```bash
# Check image pull secrets
kubectl get pod pod-name -o jsonpath='{.spec.imagePullSecrets}'
kubectl describe pod pod-name | grep -A 5 "Image"

# Test image pull
docker pull image:tag

# Check registry access
kubectl run test --image=image:tag --dry-run=client -o yaml
```

### High CPU/Memory

```bash
# Check resource usage
kubectl top pods
kubectl top pods --sort-by=memory
kubectl top pods --sort-by=cpu
kubectl top nodes

# Check limits
kubectl describe pod pod-name | grep -A 5 Limits
kubectl get pod pod-name -o jsonpath='{.spec.containers[*].resources}'

# Check node resources
kubectl describe node node-name
kubectl top node node-name
```

### Network Issues

```bash
# Test connectivity
kubectl exec pod-name -- ping service-name
kubectl exec pod-name -- nslookup service-name
kubectl exec pod-name -- wget -O- http://service-name:port

# Check service
kubectl get svc service-name
kubectl describe svc service-name
kubectl get endpoints service-name

# Check DNS
kubectl exec pod-name -- cat /etc/resolv.conf
kubectl exec pod-name -- nslookup kubernetes.default

# Network policies
kubectl get networkpolicies
kubectl describe networkpolicy policy-name
```

### Service Not Accessible

```bash
# Check service endpoints
kubectl get endpoints service-name
kubectl describe svc service-name

# Check pods
kubectl get pods -l app=label-value
kubectl get pods -l app=label-value -o wide

# Check port
kubectl get svc service-name -o jsonpath='{.spec.ports[*].port}'
kubectl get svc service-name -o jsonpath='{.spec.ports[*].targetPort}'
```

## 🐳 Docker Issues

### Container Won't Start

```bash
# View logs
docker logs container-name
docker logs container-name --tail 100
docker logs container-name --since 1h

# Inspect container
docker inspect container-name
docker inspect container-name | grep -A 10 State

# Check events
docker events
docker events --filter container=container-name
```

### Container Crashes

```bash
# View exit code
docker inspect container-name --format='{{.State.ExitCode}}'

# View logs
docker logs container-name

# Check resource limits
docker stats container-name
docker inspect container-name | grep -A 10 Resources
```

### Image Build Fails

```bash
# Build with verbose output
docker build --progress=plain -t image:tag .

# Build without cache
docker build --no-cache -t image:tag .

# Check Dockerfile syntax
docker build --dry-run -t image:tag .
```

### Network Issues

```bash
# Check network
docker network inspect network-name
docker network ls

# Test connectivity
docker exec container-name ping host
docker exec container-name nslookup host

# Check DNS
docker exec container-name cat /etc/resolv.conf
```

## 🔄 CI/CD Issues

### Pipeline Fails

```bash
# Check logs
# GitHub Actions: View workflow run
# GitLab CI: View pipeline logs
# Jenkins: View build console

# Debug mode
# GitHub Actions: Add debug step
# GitLab CI: Enable debug mode
# Jenkins: Enable verbose logging
```

### Build Timeout

```bash
# Increase timeout
# GitHub Actions: timeout-minutes
# GitLab CI: timeout
# Jenkins: timeout step
```

### Deployment Fails

```bash
# Check deployment status
kubectl rollout status deployment/deployment-name
kubectl describe deployment deployment-name

# View deployment history
kubectl rollout history deployment/deployment-name

# Rollback
kubectl rollout undo deployment/deployment-name
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

### Disk Space

```bash
# Check disk usage
df -h
du -sh /* | sort -h

# Find large files
find / -type f -size +100M 2>/dev/null
du -h / | sort -h | tail -20

# Clean up
docker system prune -a
kubectl delete pods --field-selector status.phase=Succeeded
```

## 🔍 Debugging Commands

### General Debugging

```bash
# Check system resources
htop
iostat 1
vmstat 1
sar 1

# Check logs
journalctl -f
tail -f /var/log/syslog
dmesg | tail

# Network debugging
tcpdump -i eth0
wireshark
```

### Kubernetes Debugging

```bash
# Debug pod
kubectl debug pod/pod-name -it --image=busybox
kubectl exec -it pod-name -- sh

# Port forward
kubectl port-forward pod/pod-name 8080:80

# Copy files
kubectl cp pod-name:/path/to/file ./local-file
```

### Docker Debugging

```bash
# Debug container
docker exec -it container-name sh
docker run -it --rm --entrypoint sh image:tag

# Inspect
docker inspect container-name
docker inspect image:tag

# Check logs
docker logs -f container-name
```

## 🚨 Common Solutions

### Quick Fixes

```bash
# Restart service
systemctl restart service-name
kubectl rollout restart deployment/deployment-name
docker restart container-name

# Clear cache
docker system prune
kubectl delete pods --all

# Check connectivity
ping google.com
curl -I https://google.com
```

---

**Remember**: Always check logs first, then describe resources, then check events. Use debugging tools systematically.
