# Scripting: Bash & Python for DevOps

## 🎯 Introduction

Automation is the heart of DevOps. Bash and Python are essential scripting languages for automating tasks, managing infrastructure, and building tools. This guide covers practical scripting for DevOps engineers.

## 🐚 Bash Scripting

### Basic Script Structure

```bash
#!/bin/bash
# Script description

# Set options
set -e          # Exit on error
set -u          # Exit on undefined variable
set -o pipefail # Exit on pipe failure

# Variables
NAME="DevOps"
VERSION=1.0

# Main script
echo "Hello, $NAME!"
```

### Variables

```bash
# Assignment
NAME="John"
AGE=30
PATH="/usr/bin"  # Careful: PATH is special

# Usage
echo $NAME
echo ${NAME}     # Preferred for clarity

# Default values
${VAR:-default}  # Use default if unset
${VAR:=default}  # Set default if unset
${VAR:+value}    # Use value if set
${VAR:?error}    # Error if unset
```

### Input/Output

```bash
# Read input
read -p "Enter name: " name
read -s -p "Password: " pass  # Silent input

# Output
echo "Hello, $name"
printf "Name: %s, Age: %d\n" "$name" 30

# Redirect
command > file      # Overwrite
command >> file     # Append
command 2> file     # Stderr
command &> file     # Both
command < file      # Input
```

### Conditionals

```bash
# If statement
if [ condition ]; then
    echo "True"
elif [ condition2 ]; then
    echo "Alternative"
else
    echo "False"
fi

# Test conditions
[ -f file ]         # File exists
[ -d dir ]          # Directory exists
[ -r file ]         # Readable
[ -w file ]         # Writable
[ -x file ]         # Executable
[ -z string ]       # Empty string
[ -n string ]       # Non-empty string
[ str1 = str2 ]     # Strings equal
[ str1 != str2 ]    # Strings not equal
[ num1 -eq num2 ]   # Numbers equal
[ num1 -lt num2 ]   # Less than
[ num1 -gt num2 ]   # Greater than
```

### Loops

```bash
# For loop
for i in {1..10}; do
    echo $i
done

for file in *.txt; do
    echo "Processing $file"
done

# While loop
count=0
while [ $count -lt 10 ]; do
    echo $count
    ((count++))
done

# Until loop
until [ condition ]; do
    echo "Waiting..."
    sleep 1
done
```

### Functions

```bash
# Define function
function greet() {
    local name=$1
    echo "Hello, $name!"
}

# Call function
greet "John"

# Return value
function add() {
    local a=$1
    local b=$2
    echo $((a + b))
}

result=$(add 5 3)
echo "Result: $result"
```

### Arrays

```bash
# Declare array
arr=("apple" "banana" "cherry")

# Access elements
echo ${arr[0]}      # First element
echo ${arr[@]}      # All elements
echo ${#arr[@]}     # Length

# Iterate
for item in "${arr[@]}"; do
    echo $item
done
```

### Error Handling

```bash
#!/bin/bash
set -e              # Exit on error
set -u              # Exit on undefined variable
set -o pipefail     # Exit on pipe failure

# Trap errors
trap 'echo "Error at line $LINENO"' ERR

# Check command success
if command; then
    echo "Success"
else
    echo "Failed"
    exit 1
fi
```

### Practical Examples

#### System Backup Script

```bash
#!/bin/bash
set -e

BACKUP_DIR="/backup"
SOURCE_DIR="/var/www"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/backup_$DATE.tar.gz"

# Create backup directory
mkdir -p "$BACKUP_DIR"

# Create backup
tar -czf "$BACKUP_FILE" "$SOURCE_DIR"

# Remove old backups (keep last 7 days)
find "$BACKUP_DIR" -name "backup_*.tar.gz" -mtime +7 -delete

echo "Backup created: $BACKUP_FILE"
```

#### Log Analyzer

```bash
#!/bin/bash

LOG_FILE="${1:-/var/log/app.log}"

if [ ! -f "$LOG_FILE" ]; then
    echo "Error: Log file not found"
    exit 1
fi

echo "=== Log Analysis ==="
echo "Total lines: $(wc -l < "$LOG_FILE")"
echo "Error count: $(grep -i error "$LOG_FILE" | wc -l)"
echo "Top IPs:"
grep -oE '\b([0-9]{1,3}\.){3}[0-9]{1,3}\b' "$LOG_FILE" | \
    sort | uniq -c | sort -rn | head -10
```

#### Health Check Script

```bash
#!/bin/bash

check_service() {
    local service=$1
    if systemctl is-active --quiet "$service"; then
        echo "✓ $service is running"
        return 0
    else
        echo "✗ $service is not running"
        return 1
    fi
}

check_port() {
    local host=$1
    local port=$2
    if nc -zv "$host" "$port" &>/dev/null; then
        echo "✓ Port $port on $host is open"
        return 0
    else
        echo "✗ Port $port on $host is closed"
        return 1
    fi
}

# Check services
check_service nginx
check_service mysql

# Check ports
check_port localhost 80
check_port localhost 443
```

## 🐍 Python Scripting

### Basic Script Structure

```python
#!/usr/bin/env python3
"""
Script description
"""

import sys
import os
from pathlib import Path

def main():
    """Main function"""
    print("Hello, DevOps!")

if __name__ == "__main__":
    main()
```

### Essential Libraries

```python
# Standard library
import os
import sys
import subprocess
import json
import yaml
import logging
import argparse
from pathlib import Path
from datetime import datetime

# Third-party (install with pip)
# requests - HTTP requests
# boto3 - AWS SDK
# kubernetes - K8s client
# docker - Docker client
```

### File Operations

```python
# Read file
with open('file.txt', 'r') as f:
    content = f.read()
    lines = f.readlines()

# Write file
with open('file.txt', 'w') as f:
    f.write("Hello, World!")

# Path operations
from pathlib import Path

path = Path('/var/log/app.log')
if path.exists():
    print(f"Size: {path.stat().st_size} bytes")
```

### System Operations

```python
import subprocess
import os

# Execute command
result = subprocess.run(['ls', '-la'], capture_output=True, text=True)
print(result.stdout)

# Check return code
if result.returncode == 0:
    print("Success")
else:
    print("Failed")

# Environment variables
home = os.environ.get('HOME', '/tmp')
path = os.environ.get('PATH', '')

# Change directory
os.chdir('/tmp')
```

### JSON/YAML Handling

```python
import json
import yaml

# JSON
data = {'name': 'DevOps', 'version': 1.0}
json_str = json.dumps(data)
data = json.loads(json_str)

# YAML
with open('config.yaml', 'r') as f:
    config = yaml.safe_load(f)

with open('config.yaml', 'w') as f:
    yaml.dump(config, f)
```

### Logging

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
logger.info("Information message")
logger.error("Error message")
```

### Argument Parsing

```python
import argparse

parser = argparse.ArgumentParser(description='DevOps script')
parser.add_argument('--host', required=True, help='Host name')
parser.add_argument('--port', type=int, default=80, help='Port number')
parser.add_argument('--verbose', action='store_true', help='Verbose output')

args = parser.parse_args()
print(f"Connecting to {args.host}:{args.port}")
```

### Error Handling

```python
try:
    result = 10 / 0
except ZeroDivisionError as e:
    print(f"Error: {e}")
except Exception as e:
    print(f"Unexpected error: {e}")
finally:
    print("Cleanup")
```

### Practical Examples

#### Docker Container Manager

```python
#!/usr/bin/env python3
import subprocess
import json
import sys

def list_containers():
    """List all containers"""
    result = subprocess.run(
        ['docker', 'ps', '-a', '--format', 'json'],
        capture_output=True,
        text=True
    )
    containers = []
    for line in result.stdout.strip().split('\n'):
        if line:
            containers.append(json.loads(line))
    return containers

def stop_container(container_id):
    """Stop a container"""
    result = subprocess.run(
        ['docker', 'stop', container_id],
        capture_output=True
    )
    return result.returncode == 0

def main():
    containers = list_containers()
    print(f"Found {len(containers)} containers")
    for container in containers:
        print(f"{container['ID']}: {container['Names']}")

if __name__ == "__main__":
    main()
```

#### Kubernetes Resource Checker

```python
#!/usr/bin/env python3
import subprocess
import json

def get_pods(namespace='default'):
    """Get pods in namespace"""
    cmd = ['kubectl', 'get', 'pods', '-n', namespace, '-o', 'json']
    result = subprocess.run(cmd, capture_output=True, text=True)
    if result.returncode == 0:
        return json.loads(result.stdout)
    return None

def check_pod_status(pods_data):
    """Check pod statuses"""
    if not pods_data:
        return
    
    for pod in pods_data.get('items', []):
        name = pod['metadata']['name']
        status = pod['status']['phase']
        print(f"{name}: {status}")
        
        if status != 'Running':
            print(f"  Warning: {name} is not running!")

if __name__ == "__main__":
    pods = get_pods()
    if pods:
        check_pod_status(pods)
```

#### Log Parser

```python
#!/usr/bin/env python3
import re
from collections import Counter
from pathlib import Path

def parse_log_file(log_path):
    """Parse log file and extract statistics"""
    log_path = Path(log_path)
    if not log_path.exists():
        print(f"Error: {log_path} not found")
        return
    
    ip_pattern = r'\b(?:\d{1,3}\.){3}\d{1,3}\b'
    status_pattern = r'\s(\d{3})\s'
    
    ips = []
    status_codes = []
    
    with open(log_path, 'r') as f:
        for line in f:
            # Extract IPs
            ip_match = re.search(ip_pattern, line)
            if ip_match:
                ips.append(ip_match.group())
            
            # Extract status codes
            status_match = re.search(status_pattern, line)
            if status_match:
                status_codes.append(status_match.group(1))
    
    # Statistics
    print("=== Log Statistics ===")
    print(f"Total requests: {len(ips)}")
    print(f"Unique IPs: {len(set(ips))}")
    print("\nTop 10 IPs:")
    for ip, count in Counter(ips).most_common(10):
        print(f"  {ip}: {count}")
    
    print("\nStatus Code Distribution:")
    for status, count in Counter(status_codes).most_common():
        print(f"  {status}: {count}")

if __name__ == "__main__":
    import sys
    log_file = sys.argv[1] if len(sys.argv) > 1 else '/var/log/access.log'
    parse_log_file(log_file)
```

#### AWS S3 Backup Script

```python
#!/usr/bin/env python3
import boto3
from pathlib import Path
from datetime import datetime

def backup_to_s3(local_path, bucket_name, s3_key=None):
    """Backup local directory to S3"""
    s3 = boto3.client('s3')
    local_path = Path(local_path)
    
    if not local_path.exists():
        print(f"Error: {local_path} not found")
        return False
    
    if s3_key is None:
        timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
        s3_key = f"backups/{local_path.name}_{timestamp}.tar.gz"
    
    # Create archive
    import tarfile
    archive_path = f"/tmp/{local_path.name}.tar.gz"
    with tarfile.open(archive_path, 'w:gz') as tar:
        tar.add(local_path, arcname=local_path.name)
    
    # Upload to S3
    try:
        s3.upload_file(archive_path, bucket_name, s3_key)
        print(f"Backup uploaded to s3://{bucket_name}/{s3_key}")
        return True
    except Exception as e:
        print(f"Error uploading: {e}")
        return False
    finally:
        Path(archive_path).unlink()  # Cleanup

if __name__ == "__main__":
    import sys
    if len(sys.argv) < 3:
        print("Usage: backup_s3.py <local_path> <bucket_name> [s3_key]")
        sys.exit(1)
    
    backup_to_s3(sys.argv[1], sys.argv[2], sys.argv[3] if len(sys.argv) > 3 else None)
```

## 🔧 Best Practices

### Bash Best Practices

1. **Always use `set -e`**: Exit on error
2. **Quote variables**: `"$var"` not `$var`
3. **Use `[[ ]]` instead of `[ ]`**: More features
4. **Check if files exist**: Before operations
5. **Use functions**: For reusable code
6. **Add comments**: Explain complex logic
7. **Validate input**: Check user input
8. **Handle errors**: Proper error handling

### Python Best Practices

1. **Use virtual environments**: `python -m venv venv`
2. **Follow PEP 8**: Python style guide
3. **Type hints**: For better code clarity
4. **Docstrings**: Document functions
5. **Exception handling**: Proper error handling
6. **Logging**: Use logging, not print
7. **Testing**: Write tests for scripts
8. **Dependencies**: Use requirements.txt

## 📝 Script Templates

### Bash Template

```bash
#!/bin/bash
set -euo pipefail

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
LOG_FILE="${SCRIPT_DIR}/script.log"

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

error_exit() {
    log "ERROR: $1"
    exit 1
}

main() {
    log "Starting script"
    # Your code here
    log "Script completed"
}

main "$@"
```

### Python Template

```python
#!/usr/bin/env python3
"""Script description"""

import sys
import logging
from pathlib import Path

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)

def main():
    """Main function"""
    try:
        logger.info("Starting script")
        # Your code here
        logger.info("Script completed")
    except Exception as e:
        logger.error(f"Error: {e}", exc_info=True)
        sys.exit(1)

if __name__ == "__main__":
    main()
```

## ✅ Mastery Checklist

- [ ] Write basic Bash scripts
- [ ] Handle errors in Bash
- [ ] Use functions and arrays
- [ ] Write Python scripts
- [ ] Handle files and JSON/YAML
- [ ] Use subprocess for commands
- [ ] Implement logging
- [ ] Parse command-line arguments
- [ ] Write reusable functions
- [ ] Follow best practices
- [ ] Debug scripts effectively
- [ ] Automate common tasks

---

**Remember**: Scripting is about automation and efficiency. Start simple, practice regularly, and build up to complex automation. Both Bash and Python are essential tools in your DevOps toolkit.

