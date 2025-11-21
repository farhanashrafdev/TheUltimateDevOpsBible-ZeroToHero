# Lab: Kubernetes Debugging Challenge

## 🎯 Scenario
A deployment is failing. The pods are in `CrashLoopBackOff` or `Pending`. Fix it.

## 🛠️ Setup
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: broken-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: broken
  template:
    metadata:
      labels:
        app: broken
    spec:
      containers:
      - name: app
        image: nginx:latest
        command: ["/bin/sh", "-c", "exit 1"] # Intentional crash
        resources:
          requests:
            memory: "100Gi" # Intentional scheduling failure
```

## 🕵️ Investigation Steps

### 1. Check Pod Status
```bash
kubectl get pods
# Output: Pending
```

### 2. Describe Pod (Scheduling Issue)
```bash
kubectl describe pod broken-app-xyz
# Output: "Insufficient memory"
```
**Fix**: Edit deployment, reduce memory request to "100Mi".

### 3. Check CrashLoopBackOff
```bash
kubectl get pods
# Output: CrashLoopBackOff
```

### 4. Check Logs
```bash
kubectl logs broken-app-xyz
# Output: (Empty? Or error message?)
```
**Fix**: The command `exit 1` causes immediate exit. Remove the command or fix the application logic.

### 5. Check Liveness Probe (Optional)
If pod starts but restarts after 30s:
```bash
kubectl describe pod broken-app-xyz
# Output: "Liveness probe failed: HTTP probe failed with statuscode: 500"
```

## ✅ Success Criteria
- [ ] Pod status is `Running`.
- [ ] `READY` is `1/1`.
- [ ] Logs show application started successfully.
