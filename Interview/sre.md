# SRE Interview Questions

## 🎯 Fundamentals

**Q: What is SRE and how does it differ from DevOps?**

**A:** SRE (Site Reliability Engineering) is Google's implementation of DevOps:
- DevOps is culture, SRE is practices
- SRE focuses on reliability as primary concern
- SRE uses error budgets and SLOs
- SRE applies software engineering to operations

**Q: Explain SLI, SLO, and SLA.**

**A:**
- **SLI** (Indicator): Metric measuring service level (latency, availability)
- **SLO** (Objective): Target value for SLI (99.9% availability)
- **SLA** (Agreement): Contract with consequences for missing SLO

**Q: What is an error budget?**

**A:** Error budget = 100% - SLO. Example:
- SLO: 99.9% availability
- Error budget: 0.1% (43.2 minutes/month)
- Use for deployments, experiments
- When exhausted, focus on reliability

## 📊 Monitoring & Observability

**Q: What are the four golden signals?**

**A:**
1. **Latency**: Time to serve request
2. **Traffic**: Demand on system
3. **Errors**: Rate of failed requests
4. **Saturation**: Resource utilization

**Q: How do you set up effective alerting?**

**A:**
- Alert on symptoms, not causes
- Use multi-window burn rates
- Implement alert severity levels
- Require actionable alerts with runbooks
- Avoid alert fatigue

**Q: Explain burn rate alerting.**

**A:**
```
Burn Rate = Current error rate / Allowed error rate

Example: 99.9% SLO = 0.1% error budget
- If current errors = 0.3%, burn rate = 3x
- At 3x, budget exhausts in 10 days instead of 30

Alert windows:
- 5% budget in 1 hour (fast burn) → page
- 10% budget in 6 hours → ticket
```

## 🔧 Incident Management

**Q: Walk through an incident response process.**

**A:**
1. **Detect**: Monitoring, alerts, users
2. **Respond**: Acknowledge, assemble team
3. **Triage**: Impact, urgency, escalation
4. **Mitigate**: Restore service quickly
5. **Resolve**: Fix root cause
6. **Learn**: Blameless postmortem

**Q: What makes a good postmortem?**

**A:**
- Blameless culture
- Timeline of events
- Root cause analysis (5 Whys)
- Impact measured (duration, users)
- Action items with owners
- Shared widely

## 🎯 Scenario Questions

**Q: Your service is at 99.8% but SLO is 99.9%. What do you do?**

**A:**
1. Analyze error patterns
2. Check recent deployments
3. Identify top error causes
4. Freeze feature releases
5. Focus engineering on reliability
6. Consider architecture improvements

**Q: Design an on-call rotation.**

**A:**
- Primary + secondary coverage
- 1-week rotations
- Handoff documentation
- Escalation policies
- Maximum interrupts per shift
- Compensatory time off
- Regular rotation reviews

**Q: Service is slow but no alerts fired. Why?**

**A:**
- Missing metrics for that code path
- Alert thresholds too high
- Percentile vs average mismatch
- Cold start/cache miss scenarios
- Downstream dependency latency
- Resource exhaustion not monitored

---

**Next**: Review [System Design](./system-design.md) questions.

