# On-Call & SRE

## 🎯 SRE Principles

### Service Level Objectives (SLOs)
- Target reliability
- Example: 99.9% uptime

### Service Level Indicators (SLIs)
- Measured reliability
- Example: Error rate, latency

### Error Budgets
- Acceptable failure rate
- Used for release decisions

## 📞 On-Call Practices

### Incident Response
1. **Detection**: Alert triggered
2. **Response**: On-call engineer notified
3. **Mitigation**: Quick fix applied
4. **Resolution**: Root cause fixed
5. **Post-mortem**: Learn and improve

### Runbooks
```markdown
# High CPU Usage

## Symptoms
- CPU usage > 90%
- Slow response times

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

## 🔧 Best Practices

- Clear escalation paths
- Well-documented runbooks
- Blameless post-mortems
- Proper alerting
- Regular on-call rotation
- Adequate tooling

## ✅ Key Takeaways

- SRE focuses on reliability
- SLOs define reliability targets
- Error budgets guide decisions
- On-call requires preparation
- Post-mortems drive improvement

---

**Next**: Learn production practices.

