# SRE Interview Questions

## 🎯 SRE Fundamentals

**Q: What is SRE?**
A: Site Reliability Engineering combines software engineering and operations to create scalable, reliable systems.

**Q: Explain SLO, SLI, SLA**
A: SLI (Service Level Indicator) measures reliability. SLO (Service Level Objective) is target reliability. SLA (Service Level Agreement) is commitment with consequences.

**Q: What is error budget?**
A: Acceptable failure rate (100% - SLO). Used to balance reliability and velocity.

## 📊 Reliability

**Q: Explain the four golden signals**
A: Latency (request time), Traffic (demand), Errors (failure rate), Saturation (resource utilization).

**Q: How do you measure reliability?**
A: Uptime percentage, error rate, latency percentiles (p50, p95, p99), availability metrics.

**Q: Explain incident response**
A: Detection → Response → Mitigation → Resolution → Post-mortem. Focus on blameless culture and learning.

## 📝 Scenario Questions

**Q: Design reliable system**
A: Multi-region, redundancy, health checks, auto-scaling, circuit breakers, graceful degradation, monitoring.

**Q: How do you handle on-call?**
A: Clear escalation, runbooks, monitoring, alerting, blameless post-mortems, adequate tooling, rotation.

## ✅ Key Areas

- SLOs and error budgets
- Reliability engineering
- Incident response
- Monitoring and alerting
- Capacity planning
- Toil reduction

---

**Next**: Review system design questions.

