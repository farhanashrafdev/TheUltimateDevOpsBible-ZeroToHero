# Automated Remediation: Self-Healing Systems

## 🎯 Introduction

Why wake up at 3 AM to restart a service? Automated remediation detects issues and fixes them without human intervention.

## 🔄 The Loop

1. **Detect**: Monitoring system sees an issue.
2. **Diagnose**: Verify it's a known issue.
3. **Act**: Execute remediation script.
4. **Verify**: Check if the issue is resolved.

---

## 🛠️ Tools

### 1. Kubernetes Liveness Probes (Basic)

**Restart container if it hangs**:
```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 3
  periodSeconds: 3
```

### 2. StackStorm (Advanced)

**Event-driven automation**:
- **Sensor**: Detects alert from Prometheus.
- **Rule**: If "Disk Full" AND "Severity Critical".
- **Action**: Run "Clean Logs" script.

### 3. Ansible / SaltStack

**Trigger playbooks from alerts**:
```yaml
# Remediation playbook
- name: Restart Nginx
  hosts: webservers
  tasks:
    - service:
        name: nginx
        state: restarted
```

---

## 🛡️ Safety First

**Risk**: Automation gone wrong can destroy the system (e.g., restarting all DB nodes at once).

**Safety Mechanisms**:
1. **Rate Limiting**: "Max 1 restart per hour"
2. **Maintenance Windows**: "Only remediate during business hours"
3. **Human Approval**: "Ask in Slack before deleting files"

**Example: Safe Restart Script**:
```bash
#!/bin/bash
# Check if we restarted recently
if [ -f /tmp/last_restart ]; then
  last=$(cat /tmp/last_restart)
  now=$(date +%s)
  if [ $((now - last)) -lt 3600 ]; then
    echo "Restarted too recently. Escalating to human."
    exit 1
  fi
fi

# Restart
systemctl restart myapp
date +%s > /tmp/last_restart
```

---

## 📝 Common Use Cases

1. **Disk Full**:
   - Action: Rotate logs, delete temp files, expand volume.
2. **Service Down**:
   - Action: Restart service.
3. **High Memory**:
   - Action: Restart worker process.
4. **Bad Deployment**:
   - Action: Auto-rollback.

---

## ✅ Best Practices

- [ ] Start with low-risk actions (restart stateless services)
- [ ] Implement strict rate limiting
- [ ] Log every remediation action
- [ ] Escalate to humans if remediation fails
- [ ] Test remediation scripts regularly (Game Days)

---

**Next**: [Scaling AI Workloads](./scaling-ai-workloads.md).
