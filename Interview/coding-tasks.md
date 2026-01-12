# Coding Tasks for DevOps Interviews

## 🎯 Introduction

This guide contains practical coding challenges commonly asked in DevOps interviews, covering Bash, Python, and infrastructure automation.

## 📚 Bash Scripting

### Task 1: Log Analyzer

**Problem**: Write a script to analyze nginx logs and find the top 10 IPs by request count.

```bash
#!/bin/bash
# analyze_logs.sh

LOG_FILE="${1:-/var/log/nginx/access.log}"

if [[ ! -f "$LOG_FILE" ]]; then
    echo "Error: Log file not found" >&2
    exit 1
fi

echo "Top 10 IPs by request count:"
awk '{print $1}' "$LOG_FILE" | \
    sort | \
    uniq -c | \
    sort -rn | \
    head -10 | \
    awk '{printf "%-15s %s\n", $2, $1}'
```

### Task 2: Disk Space Monitor

**Problem**: Create a monitoring script that alerts when disk usage exceeds threshold.

```bash
#!/bin/bash
# disk_monitor.sh

THRESHOLD="${1:-80}"
ALERT_EMAIL="ops@company.com"

df -h --output=pcent,target | tail -n +2 | while read usage mount; do
    usage_num="${usage%\%}"
    if (( usage_num > THRESHOLD )); then
        echo "ALERT: $mount is at $usage" | \
            mail -s "Disk Alert: $mount" "$ALERT_EMAIL"
    fi
done
```

### Task 3: Service Health Check

**Problem**: Check if multiple services are running and restart if needed.

```bash
#!/bin/bash
# health_check.sh

SERVICES=("nginx" "postgresql" "redis")

for service in "${SERVICES[@]}"; do
    if ! systemctl is-active --quiet "$service"; then
        echo "$(date): $service is down, restarting..."
        systemctl restart "$service"
        
        sleep 5
        if systemctl is-active --quiet "$service"; then
            echo "$(date): $service restarted successfully"
        else
            echo "$(date): CRITICAL - $service failed to start" >&2
        fi
    fi
done
```

## 🐍 Python Automation

### Task 4: AWS Resource Cleanup

**Problem**: Delete EC2 instances with specific tag that are older than 7 days.

```python
#!/usr/bin/env python3
import boto3
from datetime import datetime, timezone, timedelta

def cleanup_old_instances():
    ec2 = boto3.client('ec2')
    
    # Find instances with cleanup tag
    response = ec2.describe_instances(
        Filters=[
            {'Name': 'tag:Environment', 'Values': ['dev', 'test']},
            {'Name': 'instance-state-name', 'Values': ['running', 'stopped']}
        ]
    )
    
    cutoff = datetime.now(timezone.utc) - timedelta(days=7)
    instances_to_terminate = []
    
    for reservation in response['Reservations']:
        for instance in reservation['Instances']:
            if instance['LaunchTime'] < cutoff:
                instances_to_terminate.append(instance['InstanceId'])
                print(f"Marking for termination: {instance['InstanceId']}")
    
    if instances_to_terminate:
        ec2.terminate_instances(InstanceIds=instances_to_terminate)
        print(f"Terminated {len(instances_to_terminate)} instances")
    else:
        print("No instances to cleanup")

if __name__ == "__main__":
    cleanup_old_instances()
```

### Task 5: Kubernetes Pod Monitor

**Problem**: Watch pods and send Slack alert when pod enters error state.

```python
#!/usr/bin/env python3
from kubernetes import client, config, watch
import requests
import os

SLACK_WEBHOOK = os.getenv('SLACK_WEBHOOK')

def send_slack_alert(message):
    requests.post(SLACK_WEBHOOK, json={"text": message})

def watch_pods():
    config.load_incluster_config()  # or load_kube_config() for local
    v1 = client.CoreV1Api()
    
    w = watch.Watch()
    for event in w.stream(v1.list_pod_for_all_namespaces):
        pod = event['object']
        
        if pod.status.phase in ['Failed', 'Unknown']:
            msg = f"⚠️ Pod {pod.metadata.name} in {pod.metadata.namespace} is {pod.status.phase}"
            send_slack_alert(msg)
            print(msg)
        
        for status in pod.status.container_statuses or []:
            if status.state.waiting and status.state.waiting.reason == 'CrashLoopBackOff':
                msg = f"🔴 Pod {pod.metadata.name} in CrashLoopBackOff"
                send_slack_alert(msg)

if __name__ == "__main__":
    watch_pods()
```

### Task 6: YAML Config Validator

**Problem**: Validate Kubernetes YAML files for required fields.

```python
#!/usr/bin/env python3
import yaml
import sys
from pathlib import Path

REQUIRED_LABELS = ['app', 'team', 'environment']
REQUIRED_RESOURCES = ['requests', 'limits']

def validate_deployment(doc):
    errors = []
    
    # Check metadata labels
    labels = doc.get('metadata', {}).get('labels', {})
    for label in REQUIRED_LABELS:
        if label not in labels:
            errors.append(f"Missing required label: {label}")
    
    # Check container resources
    containers = doc.get('spec', {}).get('template', {}).get('spec', {}).get('containers', [])
    for container in containers:
        resources = container.get('resources', {})
        for req in REQUIRED_RESOURCES:
            if req not in resources:
                errors.append(f"Container {container['name']} missing {req}")
    
    return errors

def main():
    for path in Path('.').glob('**/*.yaml'):
        with open(path) as f:
            for doc in yaml.safe_load_all(f):
                if doc and doc.get('kind') == 'Deployment':
                    errors = validate_deployment(doc)
                    if errors:
                        print(f"\n{path}:")
                        for e in errors:
                            print(f"  ❌ {e}")

if __name__ == "__main__":
    main()
```

## 🏗️ Terraform Challenges

### Task 7: Dynamic Security Group

**Problem**: Create a security group with dynamic ingress rules.

```hcl
variable "allowed_ports" {
  type = map(object({
    port        = number
    cidr_blocks = list(string)
    description = string
  }))
  default = {
    http = {
      port        = 80
      cidr_blocks = ["0.0.0.0/0"]
      description = "HTTP access"
    }
    https = {
      port        = 443
      cidr_blocks = ["0.0.0.0/0"]
      description = "HTTPS access"
    }
  }
}

resource "aws_security_group" "dynamic" {
  name = "dynamic-sg"

  dynamic "ingress" {
    for_each = var.allowed_ports
    content {
      from_port   = ingress.value.port
      to_port     = ingress.value.port
      protocol    = "tcp"
      cidr_blocks = ingress.value.cidr_blocks
      description = ingress.value.description
    }
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

## 📋 GitHub Actions Challenges

### Task 8: Matrix Build with Caching

**Problem**: Build and test across multiple Node versions with caching.

```yaml
name: Matrix Build

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [16, 18, 20]
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      
      - name: Install and Test
        run: |
          npm ci
          npm test
      
      - name: Upload Coverage
        if: matrix.node-version == 20
        uses: codecov/codecov-action@v3
```

## ✅ Interview Tips

1. **Ask clarifying questions** before coding
2. **Explain your approach** as you code
3. **Handle errors** gracefully
4. **Write clean, readable code**
5. **Add comments** for complex logic
6. **Consider edge cases**

---

**Next**: Review [System Design](./system-design.md) questions.

