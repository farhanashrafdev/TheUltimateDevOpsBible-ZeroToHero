# Scripting: Bash & Python for DevOps Automation

> **"If you have to do it twice, automate it."**

## 🎯 Introduction

DevOps engineers are **Automation Engineers**. You glue together disparate systems using code. This guide focuses on **practical automation patterns** you'll use every day.

### What You'll Learn
- Bash for system automation
- Python for complex logic and APIs
- When to use each language
- Automation patterns and best practices
- Real-world examples

### What This Is NOT
- Not a complete language tutorial
- Not every language feature
- Not algorithm design

---

## 🐚 Bash for System Automation

### When to Use Bash
- Quick system tasks
- Piping commands together
- Shell environment manipulation
- Simple automation scripts

### Essential Bash Patterns

#### 1. Robust Script Template

```bash
#!/bin/bash
set -euo pipefail  # Exit on error, undefined var, pipe failure

# Constants
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly LOG_FILE="${SCRIPT_DIR}/script.log"

# Functions
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

error_exit() {
    log "ERROR: $1"
    exit 1
}

# Main
main() {
    log "Starting script"
    
    # Your code here
    
    log "Script completed"
}

main "$@"
```

#### 2. JSON Parsing with `jq`

`jq` is the `sed` for JSON data. **Essential for cloud/Kubernetes work.**

```bash
# Extract a value
aws s3api list-buckets | jq -r '.Buckets[].Name'

# Filter and select
kubectl get pods -o json | jq '.items[] | select(.status.phase=="Running") | .metadata.name'

# Create JSON
jq -n --arg name "DevOps" '{"hello": $name}'

# Pretty print
cat response.json | jq '.'

# Extract nested value
cat config.json | jq '.database.host'
```

**Real-world example: Get all running pod names**
```bash
kubectl get pods -o json | \
  jq -r '.items[] | select(.status.phase=="Running") | .metadata.name'
```

#### 3. Parallel Execution with `xargs`

Stop writing `for` loops. Use `xargs` for parallel execution.

```bash
# Delete all pods in parallel (max 10 at a time)
kubectl get pods -o name | xargs -P 10 -I {} kubectl delete {}

# Find and delete large files
find /var/log -name "*.log" -size +100M | xargs rm

# Process files in parallel
ls *.txt | xargs -P 4 -I {} sh -c 'process_file {}'
```

#### 4. Error Handling

```bash
#!/bin/bash
set -euo pipefail

# Trap errors
trap 'echo "Error at line $LINENO"' ERR

# Check command success
if command; then
    echo "Success"
else
    echo "Failed"
    exit 1
fi

# Check if file exists
if [ ! -f "/path/to/file" ]; then
    echo "File not found"
    exit 1
fi
```

---

## 🐍 Python for Complex Automation

### When to Use Python
- Complex logic
- API interactions
- Data processing
- Cross-platform scripts
- When you need libraries (boto3, requests, kubernetes)

### Essential Python Patterns

#### 1. Modern CLI with `typer`

Stop using `argparse`. Use `typer` for beautiful, auto-documented CLIs.

```python
import typer

app = typer.Typer()

@app.command()
def deploy(
    environment: str = typer.Option(..., help="Environment to deploy to"),
    version: str = typer.Option("latest", help="Version to deploy"),
    dry_run: bool = typer.Option(False, help="Dry run mode")
):
    """Deploy application to specified environment."""
    if dry_run:
        typer.echo(f"[DRY RUN] Would deploy {version} to {environment}")
    else:
        typer.echo(f"Deploying {version} to {environment}")
        # Actual deployment logic

if __name__ == "__main__":
    app()
```

**Usage**:
```bash
python deploy.py --environment prod --version v1.2.3
python deploy.py --help  # Auto-generated help
```

#### 2. AWS Boto3: Handling Pagination

Most AWS APIs return max 50-100 items. **You MUST handle pagination.**

```python
import boto3

s3 = boto3.client('s3')

# Wrong (only gets first page)
response = s3.list_objects_v2(Bucket='my-bucket')
for obj in response.get('Contents', []):
    print(obj['Key'])

# Correct (gets all objects)
paginator = s3.get_paginator('list_objects_v2')
for page in paginator.paginate(Bucket='my-bucket'):
    for obj in page.get('Contents', []):
        print(obj['Key'])
```

**Real-world example: Delete all objects in S3 bucket**
```python
import boto3

s3 = boto3.client('s3')
bucket = 'my-bucket'

paginator = s3.get_paginator('list_objects_v2')
for page in paginator.paginate(Bucket=bucket):
    objects = [{'Key': obj['Key']} for obj in page.get('Contents', [])]
    if objects:
        s3.delete_objects(Bucket=bucket, Delete={'Objects': objects})
```

#### 3. Data Validation with `pydantic`

Validate config files or API responses strictly.

```python
from pydantic import BaseModel, Field
from typing import List

class DatabaseConfig(BaseModel):
    host: str
    port: int = Field(default=5432, ge=1, le=65535)
    username: str
    password: str

class Config(BaseModel):
    environment: str
    replicas: int = Field(ge=1, le=100)
    database: DatabaseConfig

# Validates types automatically
config = Config(
    environment="prod",
    replicas=3,
    database={
        "host": "db.example.com",
        "port": 5432,
        "username": "admin",
        "password": "secret"
    }
)

# Access with autocomplete
print(config.database.host)
```

#### 4. Kubernetes Client

```python
from kubernetes import client, config

# Load kubeconfig
config.load_kube_config()

v1 = client.CoreV1Api()

# List pods
pods = v1.list_pod_for_all_namespaces(watch=False)
for pod in pods.items:
    print(f"{pod.metadata.namespace}/{pod.metadata.name}")

# Get pod logs
logs = v1.read_namespaced_pod_log(
    name="my-pod",
    namespace="default",
    tail_lines=100
)
print(logs)
```

---

## 🔄 Automation Patterns

### 1. Idempotency

Your script should produce the same result whether run once or 100 times.

```bash
# Bad (fails if exists)
mkdir /tmp/folder

# Good (succeeds if exists)
mkdir -p /tmp/folder

# Bad (appends every time)
echo "export PATH=$PATH:/opt/bin" >> ~/.bashrc

# Good (only adds if not present)
grep -qF 'export PATH=$PATH:/opt/bin' ~/.bashrc || \
    echo 'export PATH=$PATH:/opt/bin' >> ~/.bashrc
```

### 2. Atomic Operations

Don't leave the system in a broken state if the script crashes.

```bash
# Bad (file is incomplete if script crashes)
curl https://example.com/data > /etc/myapp/config.json

# Good (atomic move)
curl https://example.com/data > /tmp/config.json.tmp
mv /tmp/config.json.tmp /etc/myapp/config.json
```

### 3. Logging and Observability

```python
import logging

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('app.log'),
        logging.StreamHandler()
    ]
)

logger = logging.getLogger(__name__)

def deploy():
    logger.info("Starting deployment")
    try:
        # Deployment logic
        logger.info("Deployment successful")
    except Exception as e:
        logger.error(f"Deployment failed: {e}", exc_info=True)
        raise
```

---

## 🎯 Real-World Examples

### Example 1: Kubernetes Pod Cleanup Script

```bash
#!/bin/bash
set -euo pipefail

# Delete all Evicted pods
kubectl get pods --all-namespaces -o json | \
  jq -r '.items[] | select(.status.reason=="Evicted") | "\(.metadata.namespace) \(.metadata.name)"' | \
  while read namespace pod; do
    echo "Deleting $namespace/$pod"
    kubectl delete pod -n "$namespace" "$pod"
  done
```

### Example 2: AWS Resource Tagger

```python
import boto3
from typing import Dict

def tag_resources(resource_ids: list, tags: Dict[str, str], region: str = 'us-east-1'):
    """Tag AWS resources."""
    ec2 = boto3.client('ec2', region_name=region)
    
    tag_list = [{'Key': k, 'Value': v} for k, v in tags.items()]
    
    try:
        ec2.create_tags(Resources=resource_ids, Tags=tag_list)
        print(f"Tagged {len(resource_ids)} resources")
    except Exception as e:
        print(f"Error tagging resources: {e}")
        raise

# Usage
tag_resources(
    resource_ids=['i-1234567890abcdef0'],
    tags={'Environment': 'prod', 'Team': 'platform'}
)
```

### Example 3: Log Analyzer

```python
import re
from collections import Counter
from pathlib import Path

def analyze_nginx_logs(log_file: str):
    """Analyze nginx access logs."""
    ip_pattern = r'\b(?:\d{1,3}\.){3}\d{1,3}\b'
    status_pattern = r'\s(\d{3})\s'
    
    ips = []
    status_codes = []
    
    with open(log_file, 'r') as f:
        for line in f:
            # Extract IPs
            if ip_match := re.search(ip_pattern, line):
                ips.append(ip_match.group())
            
            # Extract status codes
            if status_match := re.search(status_pattern, line):
                status_codes.append(status_match.group(1))
    
    print("=== Log Analysis ===")
    print(f"Total requests: {len(ips)}")
    print(f"Unique IPs: {len(set(ips))}")
    
    print("\nTop 10 IPs:")
    for ip, count in Counter(ips).most_common(10):
        print(f"  {ip}: {count}")
    
    print("\nStatus Code Distribution:")
    for status, count in Counter(status_codes).most_common():
        print(f"  {status}: {count}")

# Usage
analyze_nginx_logs('/var/log/nginx/access.log')
```

---

## 🐹 When to Use Go?

Python is great, but Go is the language of Kubernetes, Terraform, and Docker.

**Switch to Go when**:
1. **Performance**: You need concurrency (Goroutines)
2. **Distribution**: You want a single binary (no `pip install`)
3. **Kubernetes**: You're writing a Custom Controller or Operator

**Example Go use cases**:
- Kubernetes operators
- CLI tools (kubectl, terraform)
- High-performance services

---

## ✅ Mastery Checklist

After mastering this guide, you should be able to:

- [ ] Master `jq` for JSON manipulation
- [ ] Use `set -euo pipefail` in every Bash script
- [ ] Write parallel scripts with `xargs`
- [ ] Write Python CLIs with `typer`
- [ ] Handle AWS/API pagination in Python
- [ ] Validate data with `pydantic`
- [ ] Understand idempotency
- [ ] Know when to use Bash vs Python vs Go
- [ ] Write production-ready automation scripts

---

## 📚 Practice Exercises

### Exercise 1: Bash + jq
```bash
# Get all running pods with their IPs
kubectl get pods -o json | \
  jq -r '.items[] | select(.status.phase=="Running") | "\(.metadata.name) \(.status.podIP)"'
```

### Exercise 2: Python + boto3
```python
# List all S3 buckets and their sizes
import boto3

s3 = boto3.client('s3')
cloudwatch = boto3.client('cloudwatch')

for bucket in s3.list_buckets()['Buckets']:
    # Get bucket size from CloudWatch
    response = cloudwatch.get_metric_statistics(
        Namespace='AWS/S3',
        MetricName='BucketSizeBytes',
        Dimensions=[
            {'Name': 'BucketName', 'Value': bucket['Name']},
            {'Name': 'StorageType', 'Value': 'StandardStorage'}
        ],
        StartTime=datetime.now() - timedelta(days=1),
        EndTime=datetime.now(),
        Period=86400,
        Statistics=['Average']
    )
    print(f"{bucket['Name']}: {response}")
```

---

**Next Step**: You've completed the Foundation! Now explore the **[DevOps Core](../DevOps/overview.md)** to learn Docker, Kubernetes, CI/CD, and more.

> **Remember**: Automation is about solving problems, not showing off language features. Use the simplest tool that works. Bash for simple tasks, Python for complex logic, Go for performance-critical tools.
