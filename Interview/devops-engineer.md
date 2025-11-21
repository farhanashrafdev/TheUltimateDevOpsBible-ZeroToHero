# DevOps Engineer Interview Questions & Answers

## 🎯 Introduction
This guide covers questions for Junior, Mid-level, and Senior DevOps Engineers.

## 🟢 Junior / Entry Level

### 1. What is DevOps?
**Answer**: DevOps is a culture and set of practices that combines software development (Dev) and IT operations (Ops). It aims to shorten the systems development life cycle and provide continuous delivery with high software quality. Key pillars are CAMS (Culture, Automation, Measurement, Sharing).

### 2. Explain the difference between a Virtual Machine and a Container.
**Answer**:
- **VM**: Virtualizes hardware. Runs a full OS (kernel + user space) on a hypervisor. Heavyweight, slow boot.
- **Container**: Virtualizes the OS. Shares the host kernel but has isolated user space (namespaces/cgroups). Lightweight, fast startup.

### 3. What happens when you type `ls -l` in Linux?
**Answer**:
1. Shell parses the command.
2. Checks for aliases.
3. Checks built-ins.
4. Searches `PATH` for executable `ls`.
5. `fork()`s a new process.
6. `exec()`s the `ls` binary.
7. `ls` makes syscalls (`getdents`, `stat`) to read directory and file metadata.
8. Prints output to stdout.

### 4. What is CI/CD?
**Answer**:
- **CI (Continuous Integration)**: Developers merge code to main branch frequently. Automated builds and tests run to verify changes.
- **CD (Continuous Delivery)**: Code is automatically prepared for release to production.
- **CD (Continuous Deployment)**: Code is automatically deployed to production without manual intervention.

---

## 🟡 Mid-Level

### 5. How does DNS work?
**Answer**:
1. Browser checks local cache.
2. Queries OS resolver (hosts file/cache).
3. Queries Recursive Resolver (ISP/8.8.8.8).
4. Recursive Resolver queries Root Server (.) -> TLD Server (.com) -> Authoritative Nameserver (example.com).
5. Authoritative Server returns IP.

### 6. Explain the concept of "Inode" in Linux.
**Answer**: An inode (index node) is a data structure on a filesystem that stores metadata about a file (permissions, owner, size, timestamps, pointers to data blocks). It does NOT store the filename (that's in the directory entry).

### 7. What is a "Zombie Process"?
**Answer**: A process that has completed execution (`exit()`) but still has an entry in the process table. This happens when the parent process hasn't called `wait()` to read the child's exit status.

### 8. Explain the difference between TCP and UDP.
**Answer**:
- **TCP**: Connection-oriented, reliable, ordered, flow control, congestion control. Used for HTTP, SSH, DBs.
- **UDP**: Connectionless, unreliable, unordered, fast. Used for streaming, DNS, VoIP.

---

## 🔴 Senior Level

### 9. How would you design a zero-downtime deployment strategy?
**Answer**:
- **Blue/Green**: Two identical environments. Router switches traffic from Blue (old) to Green (new). Instant rollback. Expensive (double resources).
- **Rolling Update**: Update K8s pods one by one. No downtime, less resource intensive. Slow rollback.
- **Canary**: Route 1% of traffic to new version. Monitor metrics. Gradually increase. Best for risk mitigation.

### 10. You have a "High CPU" alert on a production server. How do you debug it?
**Answer**:
1. `top` / `htop` to identify the process.
2. `pidstat -u -p <PID> 1` to see user/system time.
3. `strace -p <PID>` to see system calls (is it looping?).
4. `perf top` to see kernel/userspace functions.
5. Check logs (`journalctl`, app logs).
6. Check "Steal Time" (if VM) to see if noisy neighbor.

### 11. Explain the CAP Theorem.
**Answer**: In a distributed data store, you can only provide two of the three guarantees:
- **Consistency**: Every read receives the most recent write or an error.
- **Availability**: Every request receives a (non-error) response, without the guarantee that it contains the most recent write.
- **Partition Tolerance**: The system continues to operate despite an arbitrary number of messages being dropped (or delayed) by the network between nodes.
*Note: In distributed systems, Partition Tolerance is mandatory, so you choose between CP (Consistency) and AP (Availability).*

### 12. How does HTTPS handshake work?
**Answer**:
1. **Client Hello**: Supported cipher suites, random number.
2. **Server Hello**: Chosen cipher, server cert, random number.
3. **Key Exchange**: Client verifies cert. Generates Pre-Master Secret, encrypts with Server's Public Key.
4. **Session Keys**: Both derive symmetric Session Keys.
5. **Finished**: Encrypted communication begins.

---

## 🧠 Scenario Questions

### 13. A developer deleted the production database. What do you do?
**Answer**:
1. **Stop the bleeding**: Prevent new writes (maintenance mode).
2. **Assess**: Is it a soft delete? Is there a delayed replica?
3. **Restore**: Restore from latest backup (Point-in-Time Recovery).
4. **Verify**: Check data integrity.
5. **Resume**: Bring system back online.
6. **Post-Mortem**: Why did a dev have write access to prod DB? Implement IaC and least privilege.

### 14. The site is slow. Where do you look?
**Answer**:
- **Frontend**: Browser dev tools (Network tab), CDN hit ratio.
- **Network**: Latency, packet loss.
- **Backend**: APM (New Relic/Datadog), slow queries, CPU/Memory saturation.
- **Database**: Slow query log, lock contention, missing indexes.
