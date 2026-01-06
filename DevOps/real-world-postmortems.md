# Real-World Post-Mortems & Case Studies

## 🎯 Introduction

Learning from failures is essential in DevOps. This guide covers famous outages, their root causes, and the lessons we can apply to prevent similar issues.

## 📚 Famous Outages & Lessons

### 1. AWS S3 Outage (February 2017)

```yaml
Duration: 4+ hours
Impact: Major portion of US internet affected
Services Down: Slack, Trello, IFTTT, Imgur, many more

What Happened:
  - Engineer ran command to remove small number of S3 servers
  - Typo caused more servers to be removed than intended
  - Two critical subsystems went offline
  - Cascading failures across US-EAST-1

Timeline:
  09:37 - Command executed with incorrect input
  09:54 - S3 API error rates increased
  10:20 - AWS begins investigation
  13:18 - S3 begins recovery
  14:08 - Full recovery

Root Causes:
  1. Human error (typo in command)
  2. No rate limiting on server removal
  3. Insufficient safeguards for critical operations
  4. Single region dependencies

Fixes Implemented:
  - Added safeguards to prevent large-scale removals
  - Improved tooling with better validation
  - Faster recovery procedures
  - Better capacity monitoring

Lessons for Us:
  - Always require confirmation for destructive operations
  - Implement rate limiting on bulk changes
  - Design for multi-region redundancy
  - Test recovery procedures regularly
```

### 2. GitLab Database Deletion (January 2017)

```yaml
Duration: 18+ hours
Impact: 6 hours of data permanently lost
Data Loss: 5,037 projects, 707 users

What Happened:
  - Admin was troubleshooting replication issues late at night
  - Accidentally ran `rm -rf` on production database directory
  - Backups failed silently for multiple methods:
    - pg_dump: failed due to lack of disk space
    - LVM snapshots: never configured properly
    - Azure disk snapshots: only 24h retention
    - S3 backups: only worked for non-production

Timeline:
  23:00 - Replication issues noticed
  00:00 - Admin attempts various fixes
  00:15 - Accidentally deletes production DB directory
  00:30 - Realizes mistake, starts recovery
  06:00 - Determined 6 hours of data unrecoverable
  18:00 - Service restored

Root Causes:
  1. Five different backup methods, all failed
  2. Backups never tested for restoration
  3. Operator fatigue (working late)
  4. Confusing server naming

What They Did Right:
  - Live-streamed recovery on YouTube (transparency)
  - Published detailed post-mortem
  - Shared all findings publicly

Lessons for Us:
  - TEST YOUR BACKUPS REGULARLY
  - Don't work on production when tired
  - Clear server naming conventions
  - Multiple independent backup systems
  - Automated backup verification
```

### 3. Cloudflare Outage (July 2020)

```yaml
Duration: 27 minutes
Impact: 50% of Cloudflare's network affected
Users Impacted: Millions of websites

What Happened:
  - BGP configuration change intended for one data center
  - Configuration accidentally propagated globally
  - Caused backbone network to become unavailable

Timeline:
  21:12 - Configuration deployed to one location
  21:14 - Configuration leaked to global backbone
  21:17 - Widespread failures begin
  21:20 - Incident declared
  21:25 - Root cause identified
  21:39 - Configuration reverted
  21:42 - Service restored

Root Causes:
  1. Configuration change not properly scoped
  2. Insufficient isolation between network tiers
  3. Automated propagation without validation
  4. Missing circuit breakers

Fixes Implemented:
  - Improved configuration isolation
  - Added validation steps
  - Better change management
  - Enhanced monitoring

Lessons for Us:
  - Canary deployments for network changes
  - Blast radius reduction
  - Automated validation before propagation
  - Clear rollback procedures
```

### 4. Slack Outage (January 2021)

```yaml
Duration: 3+ hours
Impact: Global outage for all users

What Happened:
  - Routine infrastructure change (adding capacity)
  - Change triggered unexpected behavior
  - Cascading failures across multiple systems
  - Recovery complicated by scale

Root Causes:
  1. Infrastructure change had unexpected interactions
  2. Database layer became overwhelmed
  3. Cache layer failures
  4. Recovery procedures didn't scale

Lessons for Us:
  - Test infrastructure changes at scale
  - Understand system interactions
  - Have scaled recovery runbooks
  - Implement graceful degradation
```

### 5. Facebook Outage (October 2021)

```yaml
Duration: 6+ hours
Impact: Facebook, Instagram, WhatsApp globally down
Business Impact: $100M+ in lost revenue

What Happened:
  - Maintenance on backbone network
  - Configuration change disconnected FB data centers
  - BGP routes withdrawn, FB became unreachable
  - DNS servers couldn't reach backend, stopped responding
  - Even internal tools became inaccessible

Timeline:
  15:39 - Configuration change issued
  15:40 - BGP routes start withdrawing
  15:50 - Complete global outage
  16:00 - Engineers physically sent to data centers
  21:00 - BGP routes begin to recover
  22:00 - Service restored

Root Causes:
  1. Command to assess backbone capacity had bug
  2. Bug caused disconnection of all backbone
  3. BGP withdrew all routes automatically
  4. No way to access systems remotely once down

Why Recovery Was Slow:
  - Badge systems were down (couldn't enter buildings)
  - Remote access tools were down
  - Internal tools depended on Facebook infrastructure
  - DNS completely broken

Lessons for Us:
  - Out-of-band management is critical
  - Don't depend on your own service for recovery
  - Physical access procedures must work offline
  - Have separate control plane infrastructure
```

## 📋 Post-Mortem Template

### Effective Post-Mortem Structure

```markdown
# Incident Post-Mortem: [Title]

## Summary
- **Date**: YYYY-MM-DD
- **Duration**: X hours Y minutes
- **Severity**: SEV1/SEV2/SEV3
- **Author**: Name
- **Status**: Draft/Reviewed/Final

## Impact
- **User Impact**: What users experienced
- **Data Impact**: Any data loss or corruption
- **Business Impact**: Revenue, reputation, etc.
- **Affected Services**: List of services

## Timeline (All times in UTC)
| Time | Event |
|------|-------|
| HH:MM | First signs of issue |
| HH:MM | Alert fired |
| HH:MM | On-call acknowledged |
| HH:MM | Root cause identified |
| HH:MM | Fix deployed |
| HH:MM | Service restored |
| HH:MM | Monitoring confirmed stable |

## Root Cause
Technical explanation of what went wrong.

## Contributing Factors
1. Factor 1 - explanation
2. Factor 2 - explanation
3. Factor 3 - explanation

## What Went Well
- Item 1
- Item 2
- Item 3

## What Went Poorly
- Item 1
- Item 2
- Item 3

## Where We Got Lucky
- Item 1
- Item 2

## Action Items
| Priority | Action | Owner | Due Date | Status |
|----------|--------|-------|----------|--------|
| P0 | Implement monitoring | @name | Date | Open |
| P1 | Update runbook | @name | Date | Open |
| P2 | Add tests | @name | Date | Open |

## Lessons Learned
Key takeaways from this incident.

## Appendix
- Relevant logs, graphs, links
```

## 🔍 Common Failure Patterns

### Pattern 1: Cascading Failures

```yaml
Description:
  One component failure triggers failures in dependent components,
  creating a domino effect across the system.

Example:
  Database slow → App queues requests → Memory exhaustion → 
  OOM kills → More load on remaining instances → Complete outage

Prevention:
  - Circuit breakers
  - Bulkheads (isolation)
  - Graceful degradation
  - Load shedding
  - Timeouts everywhere
```

### Pattern 2: Thundering Herd

```yaml
Description:
  Many clients simultaneously retry after a failure,
  overwhelming the system when it tries to recover.

Example:
  Service goes down → Clients queue requests →
  Service comes back → All clients hit at once → Service crashes again

Prevention:
  - Exponential backoff with jitter
  - Client-side rate limiting
  - Gradual service recovery
  - Admission control
```

### Pattern 3: Configuration Drift

```yaml
Description:
  Production configuration differs from what's expected,
  causing unexpected behavior.

Example:
  Dev tested with config A → Prod has config B →
  Deployment succeeds but behavior is wrong

Prevention:
  - Infrastructure as Code
  - Configuration validation
  - Drift detection
  - Immutable infrastructure
```

### Pattern 4: Dependency Failures

```yaml
Description:
  External dependency becomes unavailable or slow,
  affecting dependent services.

Example:
  Third-party API rate limits → Requests queue →
  Timeouts → User-facing errors

Prevention:
  - Timeout all external calls
  - Cache responses where possible
  - Have fallback behavior
  - Monitor dependency health
```

## 🛡️ Prevention Strategies

### Defense in Depth

```yaml
Layer 1 - Development:
  - Code reviews
  - Unit and integration tests
  - Static analysis
  - Security scanning

Layer 2 - Deployment:
  - Canary deployments
  - Feature flags
  - Automated rollback
  - Deployment validation

Layer 3 - Runtime:
  - Health checks
  - Circuit breakers
  - Rate limiting
  - Auto-scaling

Layer 4 - Operations:
  - Monitoring and alerting
  - Runbooks
  - On-call rotation
  - Incident response

Layer 5 - Recovery:
  - Backups
  - Disaster recovery
  - Chaos engineering
  - Game days
```

### Chaos Engineering Practices

```yaml
Netflix Approach:
  - Chaos Monkey: Random instance termination
  - Chaos Kong: Region failure simulation
  - Latency Monkey: Artificial delays
  - Conformity Monkey: Non-conforming instance shutdown

Start Small:
  Week 1: Manual failover tests
  Month 1: Automated single-instance failure
  Month 3: Service-level failure injection
  Month 6: Region-level exercises
  Year 1: Full disaster simulation

Safety Rules:
  - Always have a kill switch
  - Start in non-production
  - Monitor during experiments
  - Have rollback ready
  - Limit blast radius
```

## 📊 Incident Metrics

### Key Metrics to Track

```yaml
Detection:
  MTTD (Mean Time To Detect):
    Target: < 5 minutes
    How: Better monitoring, alerting

Response:
  MTTA (Mean Time To Acknowledge):
    Target: < 15 minutes
    How: On-call processes, paging

Resolution:
  MTTR (Mean Time To Resolve):
    Target: < 1 hour (varies by severity)
    How: Runbooks, automation, expertise

Prevention:
  MTBF (Mean Time Between Failures):
    Target: Maximize
    How: Reliability engineering

Change Failure Rate:
  Target: < 15%
  How: Testing, canary deployments
```

### Incident Severity Definitions

```yaml
SEV1 - Critical:
  - Complete service outage
  - Major security breach
  - Significant data loss
  Response: All hands, 15 min acknowledgment

SEV2 - Major:
  - Partial service degradation
  - Major feature unavailable
  - Performance severely impacted
  Response: On-call + escalation, 30 min acknowledgment

SEV3 - Minor:
  - Minor feature impacted
  - Workaround available
  - Limited user impact
  Response: On-call, 1 hour acknowledgment

SEV4 - Low:
  - Cosmetic issues
  - Non-customer-facing
  - No immediate user impact
  Response: Business hours, next day
```

## ✅ Post-Mortem Best Practices

### Culture
- [ ] Blameless environment
- [ ] Focus on systems, not individuals
- [ ] Encourage honest reporting
- [ ] Share learnings widely
- [ ] Follow up on action items

### Process
- [ ] Conduct within 48-72 hours
- [ ] Include all stakeholders
- [ ] Document timeline accurately
- [ ] Identify multiple contributing factors
- [ ] Create actionable items

### Follow-Up
- [ ] Track action item completion
- [ ] Review recurring issues
- [ ] Update runbooks
- [ ] Share patterns across teams
- [ ] Measure improvement over time

---

**Next Steps**:
- Learn [Chaos Engineering](./chaos-engineering.md)
- Explore [Disaster Recovery](./disaster-recovery.md)
- Master [Monitoring & Observability](./monitoring-observability.md)

**Remember**: Every incident is an opportunity to learn. The goal isn't to prevent all failures—that's impossible. The goal is to minimize impact and recover quickly when failures occur.


