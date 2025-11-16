# On-Call & SRE: Complete Site Reliability Engineering Guide

## 🎯 Introduction

Site Reliability Engineering (SRE) is a discipline that combines software engineering and operations to build and maintain reliable systems. This guide covers SRE principles, on-call practices, and reliability engineering.

### What is SRE?

**SRE Principles**:
- **Reliability**: Systems should be reliable
- **Automation**: Automate toil away
- **Measurement**: Measure everything
- **Risk Management**: Balance speed and reliability
- **Continuous Improvement**: Learn from failures

### SRE vs Traditional Ops

| Aspect | Traditional Ops | SRE |
|--------|----------------|-----|
| **Focus** | Stability | Reliability + Velocity |
| **Approach** | Manual | Automated |
| **Metrics** | Uptime | SLIs/SLOs |
| **Failures** | Blame | Learn |
| **Changes** | Avoided | Embraced |

## 📊 SLIs, SLOs, and SLAs

### Service Level Indicators (SLIs)

**Measurable aspects of service quality**

```yaml
# Example SLIs
availability:
  definition: successful_requests / total_requests
  measurement: Prometheus query
  
latency:
  definition: p95_request_duration
  measurement: histogram_quantile(0.95, ...)
  
error_rate:
  definition: error_requests / total_requests
  measurement: rate(errors_total) / rate(requests_total)
  
throughput:
  definition: requests_per_second
  measurement: rate(requests_total[1m])
```

### Service Level Objectives (SLOs)

**Target values for SLIs**

```yaml
# Example SLOs
availability_slo:
  target: 99.9%  # 3 nines
  window: 30 days
  measurement: successful_requests / total_requests
  
latency_slo:
  target: p95 < 200ms
  window: 30 days
  measurement: histogram_quantile(0.95, ...)
  
error_rate_slo:
  target: < 0.1%
  window: 30 days
  measurement: error_rate
```

### Error Budgets

**Acceptable failure rate**

```yaml
# 99.9% availability = 0.1% error budget
# Over 30 days (43,200 minutes):
# Error budget = 43.2 minutes of downtime allowed

error_budget_calculation:
  availability: 99.9%
  window: 30 days
  total_minutes: 43200
  error_budget_minutes: 43.2
  error_budget_percentage: 0.1%
```

**Using Error Budgets**:
- **Spend on features**: If budget available
- **Freeze releases**: If budget exhausted
- **Prioritize reliability**: If budget low

### Service Level Agreements (SLAs)

**Contractual commitments**

```yaml
# Example SLA
availability_sla:
  commitment: 99.9% uptime
  penalty: Service credits if violated
  measurement: Monthly
```

## 📞 On-Call Practices

### On-Call Rotation

```yaml
# Rotation schedule
primary_oncall:
  schedule: "24/7 rotation"
  duration: "1 week"
  backup: "secondary_oncall"
  
secondary_oncall:
  role: "Backup"
  escalation: "After 15 minutes"
```

### Incident Response Process

```
1. Detection
   ├── Alert triggered
   ├── User report
   └── Monitoring alert

2. Triage
   ├── Assess severity
   ├── Assign owner
   └── Create incident

3. Investigation
   ├── Gather information
   ├── Check logs/metrics
   └── Identify root cause

4. Mitigation
   ├── Apply quick fix
   ├── Restore service
   └── Verify resolution

5. Resolution
   ├── Permanent fix
   ├── Monitor stability
   └── Close incident

6. Post-Mortem
   ├── Document incident
   ├── Identify improvements
   └── Implement changes
```

### Severity Levels

```yaml
severity_levels:
  sev1_critical:
    impact: "Service completely down"
    response_time: "15 minutes"
    resolution_time: "1 hour"
    example: "All users unable to access service"
  
  sev2_high:
    impact: "Major feature broken"
    response_time: "1 hour"
    resolution_time: "4 hours"
    example: "Payment processing down"
  
  sev3_medium:
    impact: "Minor feature broken"
    response_time: "4 hours"
    resolution_time: "1 day"
    example: "Search feature slow"
  
  sev4_low:
    impact: "Cosmetic issue"
    response_time: "1 day"
    resolution_time: "1 week"
    example: "UI text typo"
```

### Runbooks

**Step-by-step incident response guides**

```markdown
# Runbook: High CPU Usage

## Severity
Medium (SEV3)

## Symptoms
- CPU usage > 90% on multiple pods
- Slow API response times
- Timeout errors increasing
- User complaints about slowness

## Detection
- Alert: `cpu_usage > 90% for 5 minutes`
- Dashboard: Grafana CPU panel
- Logs: Slow query patterns

## Investigation Steps

### 1. Check Current State
```bash
# Check CPU usage
kubectl top pods -n production
kubectl top nodes

# Check pod status
kubectl get pods -n production
kubectl describe pod <pod-name>
```

### 2. Review Recent Changes
```bash
# Check recent deployments
kubectl rollout history deployment/myapp -n production

# Check recent events
kubectl get events -n production --sort-by=.metadata.creationTimestamp
```

### 3. Analyze Logs
```bash
# Check application logs
kubectl logs -f deployment/myapp -n production --tail=100

# Look for errors
kubectl logs deployment/myapp -n production | grep -i error
```

### 4. Check Metrics
- Review Grafana dashboards
- Check for traffic spikes
- Analyze request patterns

## Resolution Steps

### Immediate Actions
1. **Scale Up** (if needed):
   ```bash
   kubectl scale deployment myapp --replicas=10 -n production
   ```

2. **Restart Problematic Pods**:
   ```bash
   kubectl delete pod <pod-name> -n production
   ```

3. **Enable Circuit Breakers** (if available)

### Root Cause Analysis
1. Check for infinite loops in code
2. Review database query performance
3. Check for memory leaks
4. Analyze traffic patterns

### Permanent Fix
1. Fix code issues
2. Optimize database queries
3. Add caching
4. Implement rate limiting

## Prevention
- Set up HPA (Horizontal Pod Autoscaler)
- Monitor resource usage proactively
- Regular capacity planning
- Load testing before releases
- Set appropriate resource limits

## Escalation
- If not resolved in 1 hour → Escalate to team lead
- If service completely down → Escalate to manager
- If data loss suspected → Escalate immediately

## Related Runbooks
- [High Memory Usage](./runbooks/high-memory.md)
- [Database Connection Issues](./runbooks/db-connections.md)
```

## 🔧 SRE Tools

### Monitoring

```yaml
# Prometheus + Grafana
monitoring_stack:
  metrics: Prometheus
  visualization: Grafana
  alerting: AlertManager
  logging: Loki
  tracing: Jaeger
```

### Incident Management

```yaml
# PagerDuty
incident_management:
  tool: PagerDuty
  features:
    - On-call scheduling
    - Alert routing
    - Incident tracking
    - Post-mortem templates
```

### Communication

```yaml
# Slack integration
communication:
  channels:
    - "#incidents"  # Incident updates
    - "#alerts"     # Alert notifications
    - "#sre-team"   # Team communication
```

## 📈 Toil Reduction

### What is Toil?

**Toil**: Manual, repetitive, automatable work

### Automating Toil

```python
# Example: Automated health check
import subprocess
import json

def check_pod_health(namespace):
    """Automated pod health check"""
    result = subprocess.run(
        ['kubectl', 'get', 'pods', '-n', namespace, '-o', 'json'],
        capture_output=True,
        text=True
    )
    
    pods = json.loads(result.stdout)
    unhealthy = []
    
    for pod in pods['items']:
        status = pod['status']['phase']
        if status != 'Running':
            unhealthy.append({
                'name': pod['metadata']['name'],
                'status': status
            })
    
    if unhealthy:
        send_alert(f"Unhealthy pods: {unhealthy}")
    
    return len(unhealthy) == 0
```

## 🎯 SRE Best Practices

### 1. Measure Everything

```yaml
# Track:
# - SLIs
# - Error budgets
# - Toil percentage
# - Incident frequency
# - MTTR
```

### 2. Automate Toil

```yaml
# Automate:
# - Deployments
# - Health checks
# - Backups
# - Scaling
# - Remediation
```

### 3. Blameless Post-Mortems

```markdown
# Post-Mortem Template

## Incident Summary
- **Date**: 2024-01-15
- **Duration**: 2 hours
- **Impact**: 50% of users affected
- **Severity**: SEV2

## Timeline
- 10:00 AM - Alert triggered
- 10:05 AM - On-call engineer notified
- 10:15 AM - Investigation started
- 11:00 AM - Root cause identified
- 12:00 PM - Service restored

## Root Cause
Database connection pool exhausted due to:
- Increased traffic
- Slow queries
- Insufficient pool size

## Impact
- 50% of users unable to complete transactions
- Revenue impact: $10,000

## Resolution
- Increased connection pool size
- Optimized slow queries
- Added connection pool monitoring

## Action Items
- [ ] Increase connection pool size (DONE)
- [ ] Optimize slow queries (In Progress)
- [ ] Add pool monitoring (Planned)
- [ ] Load test with new pool size (Planned)

## Lessons Learned
- Need better capacity planning
- Should have caught slow queries earlier
- Connection pool monitoring is critical
```

### 4. Error Budget Policy

```yaml
error_budget_policy:
  availability: 99.9%
  window: 30 days
  actions:
    budget_available:
      - "Continue feature development"
      - "Normal release cadence"
    
    budget_low:
      - "Focus on reliability"
      - "Reduce release frequency"
    
    budget_exhausted:
      - "Freeze new features"
      - "Focus on reliability only"
      - "Review all changes"
```

## 📊 SRE Metrics

### Key Metrics

```yaml
sre_metrics:
  mttr:
    name: "Mean Time To Recovery"
    target: "< 1 hour"
    measurement: "Average time to resolve incidents"
  
  mttd:
    name: "Mean Time To Detect"
    target: "< 5 minutes"
    measurement: "Average time to detect issues"
  
  incident_frequency:
    name: "Incident Frequency"
    target: "< 1 per week"
    measurement: "Number of incidents per week"
  
  toil_percentage:
    name: "Toil Percentage"
    target: "< 50%"
    measurement: "Percentage of time spent on toil"
```

## ✅ Best Practices

### 1. Clear Escalation Paths

```yaml
escalation:
  level1: "On-call engineer"
  level2: "Team lead"
  level3: "Engineering manager"
  level4: "CTO"
```

### 2. Well-Documented Runbooks

```yaml
# Runbooks should include:
# - Symptoms
# - Investigation steps
# - Resolution steps
# - Prevention
# - Escalation
```

### 3. Blameless Post-Mortems

```yaml
# Focus on:
# - What happened
# - Why it happened
# - How to prevent
# - Not who to blame
```

### 4. Proper Alerting

```yaml
# Alert on:
# - Symptoms (user impact)
# - SLO violations
# - Not on every metric
```

### 5. Regular On-Call Rotation

```yaml
# Rotate:
# - Weekly
# - Include backup
# - Document handoff
```

## ✅ Mastery Checklist

- [ ] Understand SRE principles
- [ ] Define SLIs and SLOs
- [ ] Calculate error budgets
- [ ] Create runbooks
- [ ] Set up on-call rotation
- [ ] Implement incident response
- [ ] Conduct post-mortems
- [ ] Automate toil
- [ ] Track SRE metrics
- [ ] Continuously improve

---

**Next Steps**:
- Learn [Production Practices](./production-practices.md)
- Explore [Troubleshooting Guide](./troubleshooting-guide.md)
- Master [Monitoring & Observability](./monitoring-observability.md)

**Remember**: SRE is about balancing reliability and velocity. Use error budgets to guide decisions, automate toil, and always learn from incidents. Good SRE practices enable fast, reliable software delivery.
