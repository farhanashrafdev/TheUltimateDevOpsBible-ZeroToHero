# Disaster Recovery & Business Continuity: Complete Guide

## 🎯 Introduction

Disaster Recovery (DR) ensures business operations can continue after a catastrophic event. This guide covers strategies, implementations, and best practices for building resilient systems.

### Key Definitions

```yaml
RTO (Recovery Time Objective):
  Definition: Maximum acceptable downtime
  Example: "System must be back online within 4 hours"
  Impact: Determines DR architecture complexity

RPO (Recovery Point Objective):
  Definition: Maximum acceptable data loss
  Example: "Can lose up to 15 minutes of transactions"
  Impact: Determines backup frequency

MTTR (Mean Time To Recovery):
  Definition: Average time to restore service
  Calculation: Total downtime / Number of incidents

MTBF (Mean Time Between Failures):
  Definition: Average time between system failures
  Goal: Maximize this through reliability engineering
```

## 📊 DR Strategy Tiers

### The Four Tiers

```
┌─────────────────────────────────────────────────────────────────┐
│                     DR Strategy Comparison                       │
├──────────────────┬──────────┬──────────┬────────┬───────────────┤
│ Strategy         │ RTO      │ RPO      │ Cost   │ Complexity    │
├──────────────────┼──────────┼──────────┼────────┼───────────────┤
│ Backup & Restore │ Hours    │ Hours    │ $      │ Low           │
│ Pilot Light      │ Minutes  │ Minutes  │ $$     │ Medium        │
│ Warm Standby     │ Minutes  │ Seconds  │ $$$    │ Medium-High   │
│ Hot Standby      │ Seconds  │ Zero     │ $$$$   │ High          │
└──────────────────┴──────────┴──────────┴────────┴───────────────┘
```

### Tier 1: Backup & Restore

```yaml
Description: |
  Data is backed up and stored off-site.
  Infrastructure is recreated during recovery.

RTO: 24+ hours
RPO: 24 hours (based on backup frequency)
Cost: Lowest

Use Cases:
  - Non-critical applications
  - Development environments
  - Compliance archives

Implementation:
  - Automated backups to S3/GCS
  - Infrastructure as Code for recreation
  - Documented runbooks
```

```bash
# AWS S3 backup script
#!/bin/bash
DATE=$(date +%Y-%m-%d)
BUCKET="s3://dr-backups-${AWS_ACCOUNT_ID}"

# Backup databases
pg_dump mydb | gzip | aws s3 cp - "${BUCKET}/postgres/${DATE}/mydb.sql.gz"

# Backup files
aws s3 sync /data "${BUCKET}/files/${DATE}/"

# Backup Kubernetes resources
kubectl get all --all-namespaces -o yaml > k8s-backup.yaml
aws s3 cp k8s-backup.yaml "${BUCKET}/k8s/${DATE}/"

# Retain 30 days
aws s3 ls "${BUCKET}" | while read -r line; do
  createDate=$(echo $line | awk '{print $1}')
  if [[ $(date -d "$createDate" +%s) -lt $(date -d "30 days ago" +%s) ]]; then
    aws s3 rm "${BUCKET}/${line}" --recursive
  fi
done
```

### Tier 2: Pilot Light

```yaml
Description: |
  Minimal version of environment always running.
  Core components (DB) replicated continuously.
  Compute scaled up during disaster.

RTO: 10-30 minutes
RPO: Minutes (based on replication lag)
Cost: 10-20% of production

Use Cases:
  - Business-critical applications
  - Customer-facing services
  - E-commerce platforms

Components Always Running:
  - Database replicas
  - DNS infrastructure
  - VPN/networking
```

```hcl
# Terraform - Pilot Light Infrastructure
resource "aws_rds_cluster" "dr_replica" {
  provider               = aws.dr_region
  cluster_identifier     = "myapp-dr"
  engine                 = "aurora-postgresql"
  
  # Read replica of production
  replication_source_identifier = aws_rds_cluster.production.arn
  
  # Minimal instance for cost savings
  instance_class         = "db.t3.medium"
  
  tags = {
    Environment = "dr"
    Purpose     = "pilot-light"
  }
}

# Auto Scaling Group - 0 instances normally
resource "aws_autoscaling_group" "dr_app" {
  provider         = aws.dr_region
  name             = "myapp-dr-asg"
  min_size         = 0  # Pilot light - no instances
  max_size         = 10
  desired_capacity = 0
  
  launch_template {
    id      = aws_launch_template.dr_app.id
    version = "$Latest"
  }
}

# Scale up script for DR activation
resource "null_resource" "dr_activation_script" {
  provisioner "local-exec" {
    command = <<-EOF
      aws autoscaling update-auto-scaling-group \
        --auto-scaling-group-name myapp-dr-asg \
        --min-size 3 \
        --desired-capacity 5 \
        --region ${var.dr_region}
    EOF
  }
}
```

### Tier 3: Warm Standby

```yaml
Description: |
  Scaled-down but fully functional environment.
  All services running at minimum capacity.
  Scale up for production load during failover.

RTO: 1-10 minutes
RPO: Seconds (synchronous replication)
Cost: 30-50% of production

Use Cases:
  - Mission-critical applications
  - Financial services
  - Healthcare systems
```

```yaml
# Kubernetes - Warm Standby Configuration
# Production Cluster
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-server
  namespace: production
spec:
  replicas: 10  # Full production scale
  selector:
    matchLabels:
      app: api-server
  template:
    spec:
      containers:
      - name: api
        resources:
          requests:
            cpu: "500m"
            memory: "512Mi"
---
# DR Cluster (Warm Standby)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-server
  namespace: production
  annotations:
    dr.mode: "warm-standby"
spec:
  replicas: 2  # Minimal replicas
  selector:
    matchLabels:
      app: api-server
  template:
    spec:
      containers:
      - name: api
        resources:
          requests:
            cpu: "250m"    # Reduced resources
            memory: "256Mi"
```

### Tier 4: Hot Standby (Active-Active)

```yaml
Description: |
  Full production environment in multiple regions.
  Traffic actively distributed across regions.
  Instant failover with zero downtime.

RTO: Near zero
RPO: Zero (synchronous writes)
Cost: 100%+ of single-region production

Use Cases:
  - Zero-downtime requirements
  - Global applications
  - Trading platforms
```

```yaml
# Global Load Balancer Configuration
apiVersion: networking.gke.io/v1
kind: MultiClusterService
metadata:
  name: global-api
  namespace: production
spec:
  template:
    spec:
      selector:
        app: api-server
      ports:
      - port: 443
        targetPort: 8080
  clusters:
  - link: "us-east1/production-cluster"
  - link: "eu-west1/production-cluster"
---
# Global Traffic Policy
apiVersion: networking.gke.io/v1
kind: MultiClusterIngress
metadata:
  name: global-ingress
  annotations:
    networking.gke.io/static-ip: "global-api-ip"
spec:
  template:
    spec:
      backend:
        serviceName: global-api
        servicePort: 443
      rules:
      - http:
          paths:
          - path: /*
            backend:
              serviceName: global-api
              servicePort: 443
```

## 🔄 Database Replication Strategies

### PostgreSQL Streaming Replication

```yaml
# Primary Configuration (postgresql.conf)
wal_level: replica
max_wal_senders: 10
max_replication_slots: 10
synchronous_standby_names: 'dr_replica'

# Replica Configuration
primary_conninfo: 'host=primary.db port=5432 user=repl password=xxx'
primary_slot_name: 'dr_replica_slot'
recovery_target_timeline: 'latest'
```

```bash
# Setup streaming replication
# On replica server:
pg_basebackup -h primary.db -D /var/lib/postgresql/data \
  -U repl -v -P --wal-method=stream

# Check replication status
psql -c "SELECT client_addr, state, sent_lsn, write_lsn, replay_lsn 
         FROM pg_stat_replication;"
```

### MySQL Group Replication

```sql
-- Configure group replication
SET GLOBAL group_replication_group_name = "aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee";
SET GLOBAL group_replication_local_address = "node1:33061";
SET GLOBAL group_replication_group_seeds = "node1:33061,node2:33061,node3:33061";

-- Start replication
START GROUP_REPLICATION;

-- Check status
SELECT * FROM performance_schema.replication_group_members;
```

### AWS RDS Multi-Region

```hcl
# Primary RDS Instance
resource "aws_db_instance" "primary" {
  identifier        = "myapp-primary"
  engine            = "postgres"
  engine_version    = "14.7"
  instance_class    = "db.r5.xlarge"
  
  backup_retention_period = 7
  backup_window          = "03:00-04:00"
  
  # Enable automated backups for cross-region
  copy_tags_to_snapshot = true
}

# Cross-Region Read Replica
resource "aws_db_instance" "dr_replica" {
  provider = aws.dr_region
  
  identifier          = "myapp-dr-replica"
  replicate_source_db = aws_db_instance.primary.arn
  instance_class      = "db.r5.large"  # Can be smaller
  
  # Automated backups in DR region
  backup_retention_period = 7
}
```

## 🌐 DNS Failover Strategies

### Route53 Health Checks & Failover

```hcl
# Health Check
resource "aws_route53_health_check" "primary" {
  fqdn              = "api.primary.example.com"
  port              = 443
  type              = "HTTPS"
  resource_path     = "/health"
  failure_threshold = 3
  request_interval  = 30

  tags = {
    Name = "primary-api-health"
  }
}

# Primary Record
resource "aws_route53_record" "api_primary" {
  zone_id = aws_route53_zone.main.zone_id
  name    = "api.example.com"
  type    = "A"

  failover_routing_policy {
    type = "PRIMARY"
  }

  alias {
    name                   = aws_lb.primary.dns_name
    zone_id                = aws_lb.primary.zone_id
    evaluate_target_health = true
  }

  set_identifier  = "primary"
  health_check_id = aws_route53_health_check.primary.id
}

# Secondary/Failover Record
resource "aws_route53_record" "api_secondary" {
  zone_id = aws_route53_zone.main.zone_id
  name    = "api.example.com"
  type    = "A"

  failover_routing_policy {
    type = "SECONDARY"
  }

  alias {
    name                   = aws_lb.dr.dns_name
    zone_id                = aws_lb.dr.zone_id
    evaluate_target_health = true
  }

  set_identifier = "secondary"
}
```

### Global Load Balancing

```yaml
# Cloudflare Load Balancer
cloudflare_load_balancer:
  name: api.example.com
  fallback_pool: us-east-pool
  default_pools:
    - us-east-pool
    - eu-west-pool
  
  region_pools:
    ENAM:  # North America
      - us-east-pool
    WEUR:  # Western Europe
      - eu-west-pool
  
  pop_pools:
    LAX:  # Los Angeles
      - us-west-pool
      - us-east-pool
  
  health_check:
    path: /health
    interval: 60
    timeout: 5
    retries: 2
```

## 📋 DR Runbooks

### Failover Procedure

```yaml
# DR Failover Runbook
name: Production Failover to DR
version: 2.1
last_updated: 2024-01-15

prerequisites:
  - DR environment healthy (verify dashboard)
  - On-call engineer authorization
  - Communication channels ready

steps:
  - step: 1
    name: Declare Incident
    actions:
      - Create incident in PagerDuty
      - Notify stakeholders via Slack #incidents
      - Start incident timer
    owner: Incident Commander
    
  - step: 2
    name: Verify DR Health
    actions:
      - Check DR database replication lag
      - Verify DR application health endpoints
      - Confirm DNS TTL status
    commands:
      - "kubectl --context dr-cluster get pods -n production"
      - "psql -h dr-db -c 'SELECT pg_last_wal_replay_lsn()'"
    owner: Database Engineer
    
  - step: 3
    name: Stop Primary Writes
    actions:
      - Enable read-only mode on primary
      - Wait for replication to catch up
    commands:
      - "kubectl --context prod-cluster set env deployment/api READ_ONLY=true"
      - "sleep 30"  # Wait for replication
    owner: Platform Engineer
    
  - step: 4
    name: Promote DR Database
    actions:
      - Promote read replica to primary
      - Verify write capability
    commands:
      - "aws rds promote-read-replica --db-instance-identifier myapp-dr-replica"
      - "psql -h dr-db -c 'INSERT INTO health_check VALUES (now())'"
    owner: Database Engineer
    
  - step: 5
    name: Scale DR Application
    actions:
      - Scale application to production capacity
      - Verify all services healthy
    commands:
      - "kubectl --context dr-cluster scale deployment/api --replicas=10"
      - "kubectl --context dr-cluster rollout status deployment/api"
    owner: Platform Engineer
    
  - step: 6
    name: Switch DNS
    actions:
      - Update DNS to point to DR
      - Monitor traffic shift
    commands:
      - "aws route53 change-resource-record-sets --hosted-zone-id XXX --change-batch file://dns-failover.json"
    owner: Platform Engineer
    
  - step: 7
    name: Verify & Monitor
    actions:
      - Verify user traffic flowing to DR
      - Monitor error rates and latency
      - Update status page
    owner: Incident Commander

rollback:
  trigger: DR environment issues during failover
  steps:
    - Revert DNS changes
    - Re-enable primary writes
    - Investigate DR issues
```

### Failback Procedure

```yaml
# DR Failback Runbook
name: Failback from DR to Primary
version: 1.5

prerequisites:
  - Primary environment restored and healthy
  - Data sync from DR to Primary complete
  - Maintenance window scheduled

steps:
  - step: 1
    name: Sync Data to Primary
    actions:
      - Setup replication from DR to Primary
      - Wait for sync completion
    commands:
      - "pg_dump -h dr-db myapp | psql -h primary-db myapp"
    owner: Database Engineer
    
  - step: 2
    name: Verify Primary Ready
    actions:
      - Run full test suite against primary
      - Verify data integrity
    owner: QA Engineer
    
  - step: 3
    name: Gradual Traffic Shift
    actions:
      - Use weighted DNS (90% DR, 10% Primary)
      - Monitor error rates
      - Gradually increase Primary percentage
    owner: Platform Engineer
    
  - step: 4
    name: Complete Failback
    actions:
      - Switch 100% traffic to Primary
      - Scale down DR environment
      - Re-establish DR replication
    owner: Platform Engineer
```

## 🧪 DR Testing

### Testing Schedule

```yaml
Testing Frequency:
  Tabletop Exercises: Monthly
  Component Tests: Weekly
  Full Failover: Quarterly
  Chaos Engineering: Continuous

Test Types:
  Tabletop:
    - Walk through runbooks
    - Identify gaps
    - No actual changes
    
  Component:
    - Test backup restoration
    - Verify replication
    - DNS failover simulation
    
  Full Failover:
    - Complete DR activation
    - Run production traffic
    - Measure actual RTO/RPO
```

### Automated DR Testing

```python
#!/usr/bin/env python3
"""Automated DR Test Suite"""

import boto3
import psycopg2
import requests
import time
from datetime import datetime

class DRTest:
    def __init__(self):
        self.results = []
        
    def test_backup_restoration(self):
        """Test database backup can be restored"""
        print("Testing backup restoration...")
        
        # Get latest backup
        rds = boto3.client('rds')
        snapshots = rds.describe_db_snapshots(
            DBInstanceIdentifier='myapp-primary'
        )['DBSnapshots']
        
        latest = sorted(snapshots, key=lambda x: x['SnapshotCreateTime'])[-1]
        
        # Restore to test instance
        try:
            rds.restore_db_instance_from_db_snapshot(
                DBInstanceIdentifier='dr-test-restore',
                DBSnapshotIdentifier=latest['DBSnapshotIdentifier'],
                DBInstanceClass='db.t3.medium'
            )
            
            # Wait for availability
            waiter = rds.get_waiter('db_instance_available')
            waiter.wait(DBInstanceIdentifier='dr-test-restore')
            
            self.results.append(('backup_restoration', 'PASS', None))
            
        except Exception as e:
            self.results.append(('backup_restoration', 'FAIL', str(e)))
        finally:
            # Cleanup
            rds.delete_db_instance(
                DBInstanceIdentifier='dr-test-restore',
                SkipFinalSnapshot=True
            )
    
    def test_replication_lag(self):
        """Test replication lag is within acceptable limits"""
        print("Testing replication lag...")
        
        try:
            conn = psycopg2.connect(
                host='dr-replica.example.com',
                database='myapp',
                user='monitor'
            )
            
            cur = conn.cursor()
            cur.execute("""
                SELECT EXTRACT(EPOCH FROM (now() - pg_last_xact_replay_timestamp()))
            """)
            
            lag_seconds = cur.fetchone()[0]
            
            if lag_seconds < 60:  # Less than 1 minute
                self.results.append(('replication_lag', 'PASS', f'{lag_seconds}s'))
            else:
                self.results.append(('replication_lag', 'FAIL', f'{lag_seconds}s exceeds threshold'))
                
        except Exception as e:
            self.results.append(('replication_lag', 'FAIL', str(e)))
    
    def test_dns_failover(self):
        """Test DNS failover configuration"""
        print("Testing DNS failover...")
        
        route53 = boto3.client('route53')
        
        # Get health check status
        health_checks = route53.list_health_checks()['HealthChecks']
        
        for hc in health_checks:
            if 'primary' in hc.get('HealthCheckConfig', {}).get('FullyQualifiedDomainName', ''):
                status = route53.get_health_check_status(
                    HealthCheckId=hc['Id']
                )
                
                if status['HealthCheckObservations'][0]['StatusReport']['Status'] == 'Success':
                    self.results.append(('dns_health_check', 'PASS', None))
                else:
                    self.results.append(('dns_health_check', 'FAIL', 'Health check failing'))
    
    def test_dr_application_health(self):
        """Test DR application responds correctly"""
        print("Testing DR application health...")
        
        try:
            response = requests.get(
                'https://api.dr.example.com/health',
                timeout=10
            )
            
            if response.status_code == 200:
                self.results.append(('dr_app_health', 'PASS', None))
            else:
                self.results.append(('dr_app_health', 'FAIL', f'Status: {response.status_code}'))
                
        except Exception as e:
            self.results.append(('dr_app_health', 'FAIL', str(e)))
    
    def generate_report(self):
        """Generate test report"""
        print("\n" + "="*50)
        print("DR TEST REPORT")
        print(f"Date: {datetime.now().isoformat()}")
        print("="*50)
        
        passed = sum(1 for r in self.results if r[1] == 'PASS')
        failed = sum(1 for r in self.results if r[1] == 'FAIL')
        
        for test_name, status, details in self.results:
            symbol = "✅" if status == "PASS" else "❌"
            detail_str = f" - {details}" if details else ""
            print(f"{symbol} {test_name}: {status}{detail_str}")
        
        print(f"\nTotal: {passed} passed, {failed} failed")
        
        return failed == 0

if __name__ == '__main__':
    dr_test = DRTest()
    dr_test.test_backup_restoration()
    dr_test.test_replication_lag()
    dr_test.test_dns_failover()
    dr_test.test_dr_application_health()
    
    success = dr_test.generate_report()
    exit(0 if success else 1)
```

## ✅ DR Checklist

### Infrastructure
- [ ] Multi-region architecture designed
- [ ] Database replication configured
- [ ] Automated backups enabled
- [ ] DNS failover configured
- [ ] Load balancer health checks active

### Documentation
- [ ] Failover runbooks documented
- [ ] Failback procedures documented
- [ ] Contact lists updated
- [ ] Architecture diagrams current
- [ ] RTO/RPO documented per service

### Testing
- [ ] Backup restoration tested
- [ ] Replication lag monitored
- [ ] Full failover tested quarterly
- [ ] Runbooks validated
- [ ] Team trained on procedures

### Monitoring
- [ ] DR health dashboards
- [ ] Replication lag alerts
- [ ] Backup success alerts
- [ ] Health check alerts
- [ ] Incident communication plan

---

**Next Steps**:
- Learn [Chaos Engineering](./chaos-engineering.md)
- Explore [Monitoring & Observability](./monitoring-observability.md)
- Master [Kubernetes Operations](./kubernetes-operations.md)

**Remember**: A disaster recovery plan that isn't tested is just a wish. Regular testing is essential to ensure your DR strategy actually works when you need it.


