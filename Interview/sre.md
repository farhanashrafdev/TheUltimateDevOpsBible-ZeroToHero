# SRE (Site Reliability Engineer) Interview Questions

## 🎯 Introduction
SRE interviews focus on coding, system design, and "fixing broken things".

## 🟢 Concepts

### 1. What is the difference between DevOps and SRE?
**Answer**: "Class SRE implements interface DevOps."
- **DevOps**: Philosophy/Culture (Break silos, automate).
- **SRE**: Prescriptive way to do DevOps (SLOs, Error Budgets, Toil reduction).

### 2. Explain SLI, SLO, and SLA.
**Answer**:
- **SLI (Indicator)**: The metric (e.g., "Latency").
- **SLO (Objective)**: The goal (e.g., "99% of requests < 200ms"). Internal target.
- **SLA (Agreement)**: The contract (e.g., "If < 99%, we pay you back"). External promise.

### 3. What is "Toil"?
**Answer**: Work that is manual, repetitive, automatable, tactical, devoid of enduring value, and scales linearly as the service grows. SRE goal is to keep toil < 50% of time.

### 4. What is an Error Budget?
**Answer**: `100% - SLO`. If SLO is 99.9%, Error Budget is 0.1%.
- If budget remains: Ship features fast.
- If budget exhausted: Freeze features, focus on reliability.

---

## 🟡 Troubleshooting & Linux Internals

### 5. A server is running out of memory. Linux OOM Killer kills your database. How do you prevent this?
**Answer**:
- **Short term**: Increase swap (bad for DB performance) or add RAM.
- **Configuration**: Set `vm.overcommit_memory`.
- **OOM Score**: Adjust `/proc/<pid>/oom_score_adj` to make DB less likely to be killed (-1000).
- **Cgroups**: Limit other processes' memory usage.

### 6. What is "Load Average"?
**Answer**: The average number of processes in the run queue (running or waiting for CPU) or waiting for disk I/O (uninterruptible sleep) over 1, 5, and 15 minutes.
- High Load + Low CPU usage = Disk I/O bottleneck.

### 7. How do you troubleshoot a "Connection Refused" error?
**Answer**:
1. Is the process running? (`ps aux`)
2. Is it listening on the port? (`ss -tulpn`)
3. Is it listening on the correct interface (0.0.0.0 vs 127.0.0.1)?
4. Is a firewall blocking it? (`iptables`, `ufw`, Security Groups).
5. Is the backlog queue full?

---

## 🔴 System Design & Architecture

### 8. Design a highly available distributed key-value store.
**Answer**:
- **Partitioning**: Consistent Hashing (distribute keys across nodes).
- **Replication**: Leader-Follower or Multi-Leader.
- **Consistency**: Quorum (R + W > N).
- **Failure Detection**: Gossip Protocol.
- **Storage Engine**: LSM Tree (Write heavy) or B-Tree (Read heavy).

### 9. How do you handle "Thundering Herd" problem?
**Answer**: When many clients retry simultaneously after a service recovers, crashing it again.
- **Exponential Backoff**: Wait 1s, 2s, 4s...
- **Jitter**: Add random noise to backoff (Wait 1.1s, 2.3s...).
- **Circuit Breaker**: Stop calling failing service for a while.

### 10. Design a rate limiter.
**Answer**:
- **Algorithms**: Token Bucket, Leaky Bucket, Fixed Window, Sliding Window Log.
- **Storage**: Redis (fast, atomic counters).
- **Distributed**: Consistent hashing to route user to same limiter node.

---

## 🧠 Coding (Python/Go)

### 11. Write a script to parse a log file and find the top 5 IP addresses.
**Answer (Bash)**:
```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -5
```
**Answer (Python)**:
```python
from collections import Counter
import re

def top_ips(logfile):
    ips = []
    with open(logfile) as f:
        for line in f:
            match = re.search(r'\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}', line)
            if match:
                ips.append(match.group())
    return Counter(ips).most_common(5)
```

### 12. Implement a function to check if a binary tree is balanced.
*(Standard LeetCode style question - SREs often get these)*

---

## 🛡️ Incident Management

### 13. Describe an incident you managed.
**Structure (STAR)**:
- **Situation**: "Database CPU spiked to 100%."
- **Task**: "Restore service availability."
- **Action**: "Checked process list, found bad query, killed query, added index, scaled read replicas."
- **Result**: "Restored in 5 mins. Added query linter to CI to prevent recurrence."
