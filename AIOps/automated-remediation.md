# Automated Remediation

## 🎯 Introduction

Automated remediation enables self-healing systems that automatically respond to incidents, reducing MTTR and human toil.

## 📚 Remediation Types

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Remediation Automation Spectrum                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Manual              Semi-Auto            Fully Automated           │
│  ◄────────────────────────────────────────────────────────────►     │
│                                                                      │
│  Runbooks            Suggested            Self-Healing              │
│  Human execution     approval workflow    No intervention           │
│  High risk ok        Medium risk          Low risk only             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 🔧 Implementation Patterns

### Kubernetes Self-Healing

```yaml
# HorizontalPodAutoscaler
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app
  minReplicas: 2
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Percent
          value: 10
          periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
        - type: Percent
          value: 100
          periodSeconds: 15
```

### Event-Driven Remediation

```python
# remediation_handler.py
import boto3
from kubernetes import client, config

def handle_incident(event):
    """Route incident to appropriate remediation."""
    incident_type = event['type']
    
    handlers = {
        'pod_crash': restart_pod,
        'high_cpu': scale_deployment,
        'disk_full': cleanup_disk,
        'connection_timeout': restart_service,
    }
    
    if incident_type in handlers:
        return handlers[incident_type](event)
    else:
        create_ticket(event)  # Escalate to human

def restart_pod(event):
    """Delete crashing pod for recreation."""
    config.load_incluster_config()
    v1 = client.CoreV1Api()
    
    v1.delete_namespaced_pod(
        name=event['pod_name'],
        namespace=event['namespace']
    )
    
    log_remediation(event, 'pod_restart')

def scale_deployment(event):
    """Scale deployment based on load."""
    config.load_incluster_config()
    apps_v1 = client.AppsV1Api()
    
    current = apps_v1.read_namespaced_deployment(
        name=event['deployment'],
        namespace=event['namespace']
    )
    
    new_replicas = min(current.spec.replicas + 2, 20)
    
    apps_v1.patch_namespaced_deployment(
        name=event['deployment'],
        namespace=event['namespace'],
        body={'spec': {'replicas': new_replicas}}
    )
    
    log_remediation(event, 'scale_up', new_replicas)
```

### AWS Lambda Auto-Remediation

```python
# lambda_remediation.py
import boto3

def lambda_handler(event, context):
    """Remediate based on CloudWatch alarm."""
    alarm_name = event['alarmData']['alarmName']
    
    if 'EC2-HighCPU' in alarm_name:
        return add_instances_to_asg(event)
    elif 'EBS-HighIOPS' in alarm_name:
        return increase_ebs_iops(event)
    elif 'RDS-Connections' in alarm_name:
        return scale_rds_instance(event)

def add_instances_to_asg(event):
    autoscaling = boto3.client('autoscaling')
    
    asg_name = extract_asg_name(event)
    
    # Increase desired capacity
    response = autoscaling.set_desired_capacity(
        AutoScalingGroupName=asg_name,
        DesiredCapacity=current_capacity + 2,
        HonorCooldown=True
    )
    
    return {'status': 'scaled', 'asg': asg_name}
```

## 🛡️ Safety Controls

### Circuit Breaker Pattern

```python
class RemediationCircuitBreaker:
    def __init__(self, max_actions=5, window_minutes=30):
        self.max_actions = max_actions
        self.window_minutes = window_minutes
        self.actions = []
    
    def can_execute(self, action_type):
        """Check if remediation is allowed."""
        self.cleanup_old_actions()
        
        recent_count = len([a for a in self.actions if a['type'] == action_type])
        
        return recent_count < self.max_actions
    
    def record_action(self, action_type):
        self.actions.append({
            'type': action_type,
            'timestamp': time.time()
        })
```

### Approval Workflow

```yaml
# Argo Events + Workflows for approval
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  name: remediation-approval
spec:
  entrypoint: approval-flow
  templates:
    - name: approval-flow
      steps:
        - - name: notify
            template: send-slack
        - - name: wait-approval
            template: wait-for-approval
        - - name: execute
            template: run-remediation
            when: "{{steps.wait-approval.outputs.result}} == 'approved'"
    
    - name: wait-for-approval
      suspend:
        duration: "30m"  # Timeout
```

## 📊 Metrics & Monitoring

```yaml
# Prometheus rules for remediation tracking
groups:
  - name: remediation_metrics
    rules:
      - record: remediation:actions:total
        expr: sum(remediation_actions_total) by (type, outcome)
      
      - alert: HighRemediationRate
        expr: rate(remediation_actions_total[1h]) > 10
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High remediation action rate"
```

## ✅ Best Practices

1. **Start Simple**: Begin with low-risk, well-understood actions
2. **Implement Safeguards**: Cooldowns, limits, circuit breakers
3. **Require Approval**: For medium/high-risk actions
4. **Log Everything**: Complete audit trail
5. **Monitor Effectiveness**: Track success rates
6. **Gradual Rollout**: Enable for non-critical systems first

---

**Next**: Learn about [Scaling AI Workloads](./scaling-ai-workloads.md).

