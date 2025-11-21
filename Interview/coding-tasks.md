# Coding Interview Tasks: Complete Guide

## 🎯 Introduction

Coding interviews for DevOps roles typically focus on automation, scripting, and problem-solving. This guide covers common tasks and solutions.

## 📝 Common Tasks

### Task 1: Log Parser

**Problem**: Parse log files and extract statistics

**Requirements**:
- Parse Apache/nginx logs
- Extract IP addresses, status codes, URLs
- Calculate statistics
- Handle large files efficiently

**Solution**:
```python
import re
from collections import Counter, defaultdict
from datetime import datetime

class LogParser:
    def __init__(self, log_file):
        self.log_file = log_file
        self.ip_pattern = r'\b(\d{1,3}\.){3}\d{1,3}\b'
        self.status_pattern = r'\s(\d{3})\s'
        self.url_pattern = r'"(GET|POST|PUT|DELETE)\s+([^\s]+)'
    
    def parse(self):
        """Parse log file and extract statistics"""
        ips = []
        status_codes = []
        urls = []
        errors = []
        
        with open(self.log_file, 'r') as f:
            for line_num, line in enumerate(f, 1):
                # Extract IP
                ip_match = re.search(self.ip_pattern, line)
                if ip_match:
                    ips.append(ip_match.group())
                
                # Extract status code
                status_match = re.search(self.status_pattern, line)
                if status_match:
                    status = status_match.group(1)
                    status_codes.append(status)
                    if status.startswith('5'):
                        errors.append((line_num, line.strip()))
                
                # Extract URL
                url_match = re.search(self.url_pattern, line)
                if url_match:
                    urls.append(url_match.group(2))
        
        return {
            'total_requests': len(ips),
            'unique_ips': len(set(ips)),
            'top_ips': Counter(ips).most_common(10),
            'status_distribution': Counter(status_codes),
            'top_urls': Counter(urls).most_common(10),
            'errors': errors[:10]  # First 10 errors
        }
    
    def generate_report(self):
        """Generate formatted report"""
        stats = self.parse()
        
        report = f"""
Log Analysis Report
==================
Total Requests: {stats['total_requests']}
Unique IPs: {stats['unique_ips']}

Top 10 IPs:
"""
        for ip, count in stats['top_ips']:
            report += f"  {ip}: {count}\n"
        
        report += "\nStatus Code Distribution:\n"
        for status, count in stats['status_distribution'].most_common():
            report += f"  {status}: {count}\n"
        
        report += "\nTop 10 URLs:\n"
        for url, count in stats['top_urls']:
            report += f"  {url}: {count}\n"
        
        if stats['errors']:
            report += "\nErrors:\n"
            for line_num, line in stats['errors']:
                report += f"  Line {line_num}: {line[:100]}\n"
        
        return report

# Usage
parser = LogParser('access.log')
print(parser.generate_report())
```

### Task 2: Configuration Parser

**Problem**: Parse and validate YAML/JSON configuration

**Requirements**:
- Parse YAML/JSON files
- Validate structure
- Handle errors gracefully
- Support environment variable substitution

**Solution**:
```python
import yaml
import json
import os
import re
from typing import Dict, Any, List

class ConfigParser:
    def __init__(self, config_file):
        self.config_file = config_file
        self.config = None
    
    def load(self):
        """Load configuration file"""
        with open(self.config_file, 'r') as f:
            if self.config_file.endswith('.yaml') or self.config_file.endswith('.yml'):
                self.config = yaml.safe_load(f)
            elif self.config_file.endswith('.json'):
                self.config = json.load(f)
            else:
                raise ValueError("Unsupported file format")
        
        # Substitute environment variables
        self.config = self._substitute_env_vars(self.config)
        return self.config
    
    def _substitute_env_vars(self, obj):
        """Recursively substitute environment variables"""
        if isinstance(obj, dict):
            return {k: self._substitute_env_vars(v) for k, v in obj.items()}
        elif isinstance(obj, list):
            return [self._substitute_env_vars(item) for item in obj]
        elif isinstance(obj, str):
            # Replace ${VAR} or $VAR with environment variable
            pattern = r'\$\{?(\w+)\}?'
            def replace(match):
                var_name = match.group(1)
                return os.getenv(var_name, match.group(0))
            return re.sub(pattern, replace, obj)
        else:
            return obj
    
    def validate(self, schema: Dict[str, Any]) -> List[str]:
        """Validate configuration against schema"""
        errors = []
        
        def validate_recursive(config, schema, path=""):
            if not isinstance(schema, dict):
                return
            
            for key, expected_type in schema.items():
                current_path = f"{path}.{key}" if path else key
                
                if key not in config:
                    errors.append(f"Missing required key: {current_path}")
                    continue
                
                value = config[key]
                if isinstance(expected_type, dict):
                    if not isinstance(value, dict):
                        errors.append(f"{current_path}: Expected dict, got {type(value).__name__}")
                    else:
                        validate_recursive(value, expected_type, current_path)
                elif isinstance(expected_type, type):
                    if not isinstance(value, expected_type):
                        errors.append(f"{current_path}: Expected {expected_type.__name__}, got {type(value).__name__}")
        
        validate_recursive(self.config, schema)
        return errors
    
    def get(self, key_path: str, default=None):
        """Get value by dot-separated key path"""
        keys = key_path.split('.')
        value = self.config
        
        for key in keys:
            if isinstance(value, dict) and key in value:
                value = value[key]
            else:
                return default
        
        return value

# Usage
parser = ConfigParser('config.yaml')
config = parser.load()

schema = {
    'database': {
        'host': str,
        'port': int,
        'name': str
    },
    'api': {
        'timeout': int,
        'retries': int
    }
}

errors = parser.validate(schema)
if errors:
    print("Validation errors:")
    for error in errors:
        print(f"  - {error}")
else:
    print("Configuration valid!")
    print(f"Database host: {parser.get('database.host')}")
```

### Task 3: API Client with Retries

**Problem**: Create robust API client with error handling

**Requirements**:
- HTTP client with retries
- Exponential backoff
- Timeout handling
- Error classification
- Logging

**Solution**:
```python
import requests
import time
import logging
from typing import Optional, Dict, Any
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

class APIClient:
    def __init__(self, base_url: str, timeout: int = 30, max_retries: int = 3):
        self.base_url = base_url
        self.timeout = timeout
        self.max_retries = max_retries
        self.session = requests.Session()
        
        # Configure retries
        retry_strategy = Retry(
            total=max_retries,
            backoff_factor=1,
            status_forcelist=[429, 500, 502, 503, 504],
            allowed_methods=["GET", "POST", "PUT", "DELETE"]
        )
        
        adapter = HTTPAdapter(max_retries=retry_strategy)
        self.session.mount("http://", adapter)
        self.session.mount("https://", adapter)
        
        # Setup logging
        logging.basicConfig(level=logging.INFO)
        self.logger = logging.getLogger(__name__)
    
    def request(self, method: str, endpoint: str, **kwargs) -> Optional[Dict[Any, Any]]:
        """Make API request with retries"""
        url = f"{self.base_url}/{endpoint.lstrip('/')}"
        
        try:
            response = self.session.request(
                method=method,
                url=url,
                timeout=self.timeout,
                **kwargs
            )
            response.raise_for_status()
            return response.json()
        
        except requests.exceptions.Timeout:
            self.logger.error(f"Request timeout: {url}")
            return None
        
        except requests.exceptions.ConnectionError:
            self.logger.error(f"Connection error: {url}")
            return None
        
        except requests.exceptions.HTTPError as e:
            self.logger.error(f"HTTP error {e.response.status_code}: {url}")
            return None
        
        except requests.exceptions.RequestException as e:
            self.logger.error(f"Request error: {e}")
            return None
    
    def get(self, endpoint: str, **kwargs) -> Optional[Dict[Any, Any]]:
        """GET request"""
        return self.request('GET', endpoint, **kwargs)
    
    def post(self, endpoint: str, data: Dict[Any, Any], **kwargs) -> Optional[Dict[Any, Any]]:
        """POST request"""
        return self.request('POST', endpoint, json=data, **kwargs)
    
    def put(self, endpoint: str, data: Dict[Any, Any], **kwargs) -> Optional[Dict[Any, Any]]:
        """PUT request"""
        return self.request('PUT', endpoint, json=data, **kwargs)
    
    def delete(self, endpoint: str, **kwargs) -> Optional[Dict[Any, Any]]:
        """DELETE request"""
        return self.request('DELETE', endpoint, **kwargs)

# Usage
client = APIClient('https://api.example.com', timeout=30, max_retries=3)
response = client.get('/users/123')
if response:
    print(response)
```

### Task 4: Deployment Script

**Problem**: Script to deploy application with health checks

**Requirements**:
- Deploy application
- Health checks
- Rollback on failure
- Logging
- Idempotent

**Solution**:
```python
import subprocess
import time
import sys
import logging
from typing import Optional

class DeploymentScript:
    def __init__(self, app_name: str, health_check_url: str):
        self.app_name = app_name
        self.health_check_url = health_check_url
        self.previous_version = None
        
        logging.basicConfig(
            level=logging.INFO,
            format='%(asctime)s - %(levelname)s - %(message)s'
        )
        self.logger = logging.getLogger(__name__)
    
    def get_current_version(self) -> Optional[str]:
        """Get current deployed version"""
        try:
            result = subprocess.run(
                ['kubectl', 'get', 'deployment', self.app_name, '-o', 'jsonpath={.spec.template.spec.containers[0].image}'],
                capture_output=True,
                text=True,
                check=True
            )
            return result.stdout.strip()
        except subprocess.CalledProcessError:
            return None
    
    def deploy(self, new_image: str) -> bool:
        """Deploy new version"""
        self.logger.info(f"Starting deployment of {new_image}")
        
        # Save current version for rollback
        self.previous_version = self.get_current_version()
        self.logger.info(f"Current version: {self.previous_version}")
        
        try:
            # Update deployment
            subprocess.run(
                ['kubectl', 'set', 'image', f'deployment/{self.app_name}', 
                 f'{self.app_name}={new_image}'],
                check=True
            )
            self.logger.info("Deployment updated")
            
            # Wait for rollout
            subprocess.run(
                ['kubectl', 'rollout', 'status', f'deployment/{self.app_name}'],
                check=True,
                timeout=300
            )
            self.logger.info("Rollout completed")
            
            # Health check
            if self.health_check():
                self.logger.info("Health check passed")
                return True
            else:
                self.logger.error("Health check failed, rolling back")
                self.rollback()
                return False
        
        except subprocess.TimeoutExpired:
            self.logger.error("Deployment timeout, rolling back")
            self.rollback()
            return False
        
        except subprocess.CalledProcessError as e:
            self.logger.error(f"Deployment failed: {e}")
            self.rollback()
            return False
    
    def health_check(self, max_attempts: int = 10, interval: int = 5) -> bool:
        """Perform health check"""
        import requests
        
        for attempt in range(max_attempts):
            try:
                response = requests.get(self.health_check_url, timeout=5)
                if response.status_code == 200:
                    return True
            except requests.RequestException:
                pass
            
            if attempt < max_attempts - 1:
                self.logger.info(f"Health check attempt {attempt + 1} failed, retrying...")
                time.sleep(interval)
        
        return False
    
    def rollback(self):
        """Rollback to previous version"""
        if not self.previous_version:
            self.logger.error("No previous version to rollback to")
            return
        
        self.logger.info(f"Rolling back to {self.previous_version}")
        try:
            subprocess.run(
                ['kubectl', 'rollout', 'undo', f'deployment/{self.app_name}'],
                check=True
            )
            subprocess.run(
                ['kubectl', 'rollout', 'status', f'deployment/{self.app_name}'],
                check=True,
                timeout=300
            )
            self.logger.info("Rollback completed")
        except subprocess.CalledProcessError as e:
            self.logger.error(f"Rollback failed: {e}")
            sys.exit(1)

# Usage
if __name__ == '__main__':
    deployer = DeploymentScript('my-app', 'http://my-app/health')
    success = deployer.deploy('my-app:v2.0.0')
    sys.exit(0 if success else 1)
```

### Task 5: Monitoring Script

**Problem**: Monitor system resources and alert

**Requirements**:
- Monitor CPU, memory, disk
- Alert on thresholds
- Log metrics
- Configurable thresholds

**Solution**:
```python
import psutil
import time
import logging
import json
from datetime import datetime
from typing import Dict, Any

class SystemMonitor:
    def __init__(self, thresholds: Dict[str, float]):
        self.thresholds = thresholds
        self.alerts = []
        
        logging.basicConfig(
            level=logging.INFO,
            format='%(asctime)s - %(levelname)s - %(message)s'
        )
        self.logger = logging.getLogger(__name__)
    
    def get_metrics(self) -> Dict[str, Any]:
        """Collect system metrics"""
        return {
            'cpu_percent': psutil.cpu_percent(interval=1),
            'memory_percent': psutil.virtual_memory().percent,
            'disk_percent': psutil.disk_usage('/').percent,
            'timestamp': datetime.now().isoformat()
        }
    
    def check_thresholds(self, metrics: Dict[str, Any]) -> list:
        """Check metrics against thresholds"""
        alerts = []
        
        if metrics['cpu_percent'] > self.thresholds.get('cpu', 80):
            alerts.append({
                'metric': 'cpu',
                'value': metrics['cpu_percent'],
                'threshold': self.thresholds['cpu'],
                'severity': 'critical' if metrics['cpu_percent'] > 90 else 'warning'
            })
        
        if metrics['memory_percent'] > self.thresholds.get('memory', 80):
            alerts.append({
                'metric': 'memory',
                'value': metrics['memory_percent'],
                'threshold': self.thresholds['memory'],
                'severity': 'critical' if metrics['memory_percent'] > 90 else 'warning'
            })
        
        if metrics['disk_percent'] > self.thresholds.get('disk', 80):
            alerts.append({
                'metric': 'disk',
                'value': metrics['disk_percent'],
                'threshold': self.thresholds['disk'],
                'severity': 'critical' if metrics['disk_percent'] > 90 else 'warning'
            })
        
        return alerts
    
    def monitor(self, interval: int = 60, duration: int = 3600):
        """Monitor system for specified duration"""
        start_time = time.time()
        
        while time.time() - start_time < duration:
            metrics = self.get_metrics()
            alerts = self.check_thresholds(metrics)
            
            # Log metrics
            self.logger.info(f"Metrics: {json.dumps(metrics)}")
            
            # Handle alerts
            if alerts:
                for alert in alerts:
                    self.logger.warning(
                        f"ALERT: {alert['metric']} = {alert['value']}% "
                        f"(threshold: {alert['threshold']}%, severity: {alert['severity']})"
                    )
                    self.send_alert(alert)
            
            time.sleep(interval)
    
    def send_alert(self, alert: Dict[str, Any]):
        """Send alert (implement based on requirements)"""
        # Example: Send to alerting system
        # In real implementation: PagerDuty, Slack, email, etc.
        self.alerts.append(alert)

# Usage
thresholds = {
    'cpu': 80,
    'memory': 85,
    'disk': 90
}

monitor = SystemMonitor(thresholds)
monitor.monitor(interval=60, duration=3600)
```

## ✅ Preparation Tips

1. **Practice Python/Bash**: Common languages for DevOps
2. **Data Structures**: Lists, dicts, sets
3. **Error Handling**: Try-except, logging
4. **File I/O**: Reading/writing files
5. **Regular Expressions**: Pattern matching
6. **APIs**: HTTP requests, REST APIs
7. **System Commands**: subprocess, os
8. **Testing**: Write testable code
9. **Documentation**: Clear comments
10. **Code Quality**: Clean, readable code

## 🎯 Key Skills

- [ ] Python scripting
- [ ] Bash scripting
- [ ] File processing
- [ ] API integration
- [ ] Error handling
- [ ] Logging
- [ ] Testing
- [ ] Code organization
- [ ] Problem-solving
- [ ] Communication

---

**Remember**: Coding interviews test problem-solving and coding ability. Write clean, readable code. Explain your approach. Handle edge cases. Test your solutions. Show good coding practices.
