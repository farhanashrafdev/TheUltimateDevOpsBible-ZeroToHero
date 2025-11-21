# SRE Interview Questions: Complete Guide

## 🎯 Introduction

This comprehensive guide covers Site Reliability Engineering (SRE) interview questions from fundamentals to advanced reliability engineering.

## 📚 SRE Fundamentals

### What is SRE?

**Answer**: Site Reliability Engineering combines software engineering and operations to create scalable, reliable systems:

**Key Principles**:
1. **Automation**: Automate toil away
2. **Measurement**: Measure everything
3. **Risk Management**: Balance speed and reliability
4. **Continuous Improvement**: Learn from failures
5. **Shared Ownership**: Everyone owns reliability

**SRE vs Traditional Ops**:
- **Ops**: Manual, reactive, stability-focused
- **SRE**: Automated, proactive, reliability + velocity

### Explain SLO, SLI, SLA

**Answer**:

**SLI (Service Level Indicator)**:
- Measurable aspect of service quality
- Examples: Availability, latency, error rate
- Formula: `SLI = (Good Events / Total Events) * 100`

**SLO (Service Level Objective)**:
- Target value for SLI
- Example: 99.9% availability
- Used for decision-making

**SLA (Service Level Agreement)**:
- Contractual commitment
- Includes consequences if violated
- Example: Service credits

**Example**:
```
SLI: Successful requests / Total requests
SLO: 99.9% (3 nines)
SLA: 99.9% uptime or service credits
```

### What is Error Budget?

**Answer**: Error budget = 100% - SLO

**Example**:
- SLO: 99.9% availability
- Error Budget: 0.1% (43.2 minutes per month)

**How it works**:
- **Budget Available**: Continue feature development
- **Budget Low**: Focus on reliability
- **Budget Exhausted**: Freeze new features, focus on reliability

**Purpose**: Balance velocity and reliability

## 📊 Reliability Metrics

### Explain the Four Golden Signals

**Answer**:

1. **Latency**: Time to serve a request
   - Focus on tail latency (p95, p99)
   - Differentiate successful vs failed requests

2. **Traffic**: Demand on system
   - Requests per second
   - Concurrent users
   - Data transfer rate

3. **Errors**: Rate of failed requests
   - HTTP 5xx errors
   - Failed requests
   - Error rate percentage

4. **Saturation**: Resource utilization
   - CPU, memory, disk, network
   - Queue depth
   - Resource exhaustion

**Why These?**: They indicate system health and user experience

### How do you measure reliability?

**Answer**:

**Availability**:
- Uptime percentage
- Formula: `(Uptime / Total Time) * 100`
- Example: 99.9% = 3 nines

**Error Rate**:
- Failed requests / Total requests
- Track by endpoint, service, region

**Latency**:
- Percentiles: p50, p95, p99
- Average latency
- Tail latency (p99)

**MTTR (Mean Time to Recovery)**:
- Average time to recover from failures
- Target: < 1 hour

**MTTD (Mean Time to Detect)**:
- Average time to detect issues
- Target: < 5 minutes

## 🔧 Reliability Engineering

### How do you design a reliable system?

**Answer**:

1. **Redundancy**:
   - Multi-region deployment
   - Multiple availability zones
   - Replicated databases

2. **Health Checks**:
   - Liveness probes
   - Readiness probes
   - Startup probes

3. **Auto-Scaling**:
   - Horizontal Pod Autoscaler
   - Cluster Autoscaler
   - Predictive scaling

4. **Circuit Breakers**:
   - Prevent cascade failures
   - Fail fast
   - Automatic recovery

5. **Graceful Degradation**:
   - Fallback mechanisms
   - Reduced functionality
   - User experience preservation

6. **Monitoring**:
   - Comprehensive observability
   - Real-time alerts
   - Dashboards

7. **Disaster Recovery**:
   - Backup strategy
   - Recovery procedures
   - Regular testing

### Explain Incident Response

**Answer**: Structured approach to handling incidents:

**Stages**:
1. **Detection**: Identify issue (alerts, monitoring, user reports)
2. **Response**: On-call engineer notified
3. **Triage**: Assess severity and impact
4. **Investigation**: Gather information, identify root cause
5. **Mitigation**: Apply quick fix to restore service
6. **Resolution**: Implement permanent fix
7. **Post-Mortem**: Document, learn, improve

**Best Practices**:
- Blameless culture
- Clear escalation paths
- Well-documented runbooks
- Communication plan
- Post-mortem within 48 hours

### What is a Runbook?

**Answer**: Step-by-step guide for common operations:

**Components**:
- **Symptoms**: What indicates the issue
- **Investigation**: How to diagnose
- **Resolution**: How to fix
- **Prevention**: How to prevent recurrence
- **Escalation**: When to escalate

**Example**:
```markdown
# High CPU Usage

## Symptoms
- CPU usage > 90%
- Slow response times
- Timeout errors

## Investigation
1. Check top processes: `top`
2. Review application logs
3. Check recent deployments

## Resolution
1. Scale up if needed
2. Restart problematic services
3. Check for infinite loops

## Prevention
- Set up autoscaling
- Monitor resource usage
- Regular capacity planning
```

## 📈 Capacity Planning

### How do you do capacity planning?

**Answer**:

1. **Measure Current Usage**:
   - Resource utilization
   - Traffic patterns
   - Growth trends

2. **Forecast Demand**:
   - Historical trends
   - Business projections
   - Seasonal patterns

3. **Calculate Capacity Needs**:
   - Headroom (20-30%)
   - Peak capacity
   - Growth buffer

4. **Plan Scaling**:
   - Horizontal scaling
   - Vertical scaling
   - Auto-scaling configuration

5. **Monitor and Adjust**:
   - Track actual vs forecast
   - Adjust forecasts
   - Update capacity plans

### Explain Toil

**Answer**: Toil is manual, repetitive, automatable work:

**Characteristics**:
- Manual
- Repetitive
- Automatable
- No enduring value
- Grows with system

**Examples**:
- Manual deployments
- Restarting services
- Checking logs manually
- Creating tickets

**Reduction**:
- Automate repetitive tasks
- Self-service tools
- Eliminate root causes
- Target: < 50% of time on toil

## 🎯 Scenario Questions

### How do you handle on-call?

**Answer**:

1. **Preparation**:
   - Clear runbooks
   - Access to systems
   - Escalation paths
   - Communication channels

2. **During Incident**:
   - Follow runbooks
   - Communicate status
   - Document actions
   - Escalate when needed

3. **After Incident**:
   - Post-mortem
   - Update runbooks
   - Improve monitoring
   - Automate fixes

4. **Best Practices**:
   - Rotation schedule
   - Backup on-call
   - Adequate tooling
   - Blameless culture

### Design a monitoring system

**Answer**:

1. **Data Collection**:
   - Metrics (Prometheus)
   - Logs (ELK, Loki)
   - Traces (Jaeger)

2. **Storage**:
   - Time-series database
   - Log aggregation
   - Long-term retention

3. **Alerting**:
   - AlertManager
   - Alert routing
   - On-call integration

4. **Visualization**:
   - Grafana dashboards
   - Service health pages
   - Custom dashboards

5. **Intelligence**:
   - Anomaly detection
   - Alert correlation
   - Root cause analysis

### How do you reduce alert fatigue?

**Answer**:

1. **Alert Tuning**:
   - Set appropriate thresholds
   - Reduce noise
   - Focus on symptoms

2. **Alert Correlation**:
   - Group related alerts
   - Root cause alerts
   - Suppress noise

3. **Intelligent Alerting**:
   - ML-based filtering
   - Context-aware alerts
   - Priority ranking

4. **Alert Policies**:
   - Alert on SLO violations
   - Alert on user impact
   - Don't alert on everything

5. **Review and Improve**:
   - Regular alert reviews
   - Remove unused alerts
   - Update thresholds

## ✅ Key Areas to Master

- [ ] SRE principles and culture
- [ ] SLOs, SLIs, and error budgets
- [ ] Reliability engineering
- [ ] Incident response
- [ ] Monitoring and alerting
- [ ] Capacity planning
- [ ] Toil reduction
- [ ] Post-mortems
- [ ] On-call practices
- [ ] System design for reliability

---

**Remember**: SRE interviews test both technical skills and cultural fit. Be prepared to discuss reliability engineering, incident response, and how you balance velocity with reliability. Show understanding of SRE principles and practices.
