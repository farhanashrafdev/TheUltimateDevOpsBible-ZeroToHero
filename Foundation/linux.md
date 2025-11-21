# Linux Fundamentals for DevOps: Production Skills

> **"Everything is a file."** - The Unix Philosophy

## 🎯 Introduction

Linux is the operating system of the cloud. **95% of servers run Linux**. As a DevOps engineer, you don't just "use" Linux—you must understand how it works, how to troubleshoot it, and how to optimize it for production workloads.

This guide focuses on **practical Linux skills you'll use every day** in a DevOps role.

### What You'll Learn
- Essential commands for daily work
- System administration and troubleshooting
- Performance analysis and optimization
- Security hardening
- Automation with shell scripting

### What This Is NOT
- Not a Linux certification exam prep
- Not kernel development
- Not every obscure command flag

---

## 🧠 Linux Architecture: How It Works

### User Space vs. Kernel Space

```
┌─────────────────────────────────────┐
│        User Space                    │
│  ┌──────────┐  ┌──────────┐         │
│  │  nginx   │  │  python  │  Apps   │
│  └──────────┘  └──────────┘         │
│         ↕ System Calls ↕             │
├─────────────────────────────────────┤
│        Kernel Space                  │
│  ┌──────────────────────────────┐   │
│  │   Linux Kernel               │   │
│  │  - Process Management        │   │
│  │  - Memory Management         │   │
│  │  - File System               │   │
│  │  - Network Stack             │   │
│  └──────────────────────────────┘   │
├─────────────────────────────────────┤
│        Hardware                      │
│  CPU | Memory | Disk | Network      │
└─────────────────────────────────────┘
```

**Key Concept**: Applications in User Space cannot directly access hardware. They must use **system calls** to ask the kernel.

### The Boot Process (Troubleshooting Boot Failures)

1. **BIOS/UEFI**: Hardware initialization, POST (Power-On Self-Test)
2. **Bootloader (GRUB)**: Loads the kernel into memory
3. **Kernel**: Mounts the root filesystem (initramfs)
4. **Init System (systemd)**: PID 1, starts all other services
5. **User Space**: Login prompt / GUI

**Common boot issues**:
```bash
# Check boot logs
journalctl -b

# Check kernel messages
dmesg | less

# Check systemd services that failed
systemctl --failed
```

---

## 📂 File System Essentials

### The /proc and /sys Filesystems

These are **pseudo-filesystems** that provide an interface to kernel data.

#### /proc - Process Information
```bash
# CPU information
cat /proc/cpuinfo

# Memory information
cat /proc/meminfo

# Current processes
ls /proc/
# Each number is a PID

# Process details
cat /proc/1234/status
cat /proc/1234/cmdline
```

#### /sys - Device and Driver Information
```bash
# Block devices
ls /sys/block/

# Network interfaces
ls /sys/class/net/
```

**Real-world use case**:
```bash
# Check if a process is still running
if [ -d "/proc/$PID" ]; then
    echo "Process is running"
fi
```

### File System Hierarchy (What Goes Where)

| Directory | Purpose | Example |
|:---|:---|:---|
| `/bin`, `/usr/bin` | Essential binaries | `ls`, `cat`, `grep` |
| `/sbin`, `/usr/sbin` | System binaries | `iptables`, `systemctl` |
| `/etc` | Configuration files | `/etc/nginx/nginx.conf` |
| `/var` | Variable data | `/var/log/`, `/var/lib/docker/` |
| `/tmp` | Temporary files | Cleared on reboot |
| `/home` | User home directories | `/home/username/` |
| `/opt` | Optional software | `/opt/myapp/` |
| `/proc` | Process information | Pseudo-filesystem |
| `/sys` | Device information | Pseudo-filesystem |

---

## 🛠️ Essential Commands for Daily Work

### File Operations (The Basics Done Right)

```bash
# List files (with useful flags)
ls -lah                    # Long format, all files, human-readable sizes
ls -lht                    # Sorted by time (newest first)
ls -lhS                    # Sorted by size (largest first)

# Find files (powerful search)
find /var/log -name "*.log" -mtime -7    # Modified in last 7 days
find /var/log -type f -size +100M        # Files larger than 100MB
find . -name "*.tmp" -delete             # Find and delete

# Search in files (grep)
grep -r "ERROR" /var/log/                # Recursive search
grep -i "error" file.log                 # Case-insensitive
grep -v "INFO" file.log                  # Invert match (exclude INFO)
grep -A 5 "ERROR" file.log               # Show 5 lines after match
grep -B 5 "ERROR" file.log               # Show 5 lines before match
```

### Text Processing (DevOps Essentials)

```bash
# View files
cat file.txt               # Display entire file
less file.txt              # Page through file (q to quit)
head -n 20 file.txt        # First 20 lines
tail -n 20 file.txt        # Last 20 lines
tail -f /var/log/app.log   # Follow (watch) file in real-time

# Text manipulation
cut -d: -f1 /etc/passwd    # Extract first field (usernames)
awk '{print $1}' file      # Print first column
sed 's/old/new/g' file     # Replace text
sort file                  # Sort lines
uniq file                  # Remove duplicates
wc -l file                 # Count lines
```

**Real-world example: Extract unique IPs from access log**
```bash
# access.log format: IP - - [timestamp] "GET /path" 200
awk '{print $1}' access.log | sort | uniq | wc -l
```

### Process Management (Production Debugging)

```bash
# View processes
ps aux                     # All processes
ps aux | grep nginx        # Filter processes
top                        # Interactive process viewer
htop                       # Better top (install: apt install htop)

# Process control
kill PID                   # Graceful kill (SIGTERM)
kill -9 PID                # Force kill (SIGKILL)
killall nginx              # Kill by name
pkill -f "python app.py"   # Kill by pattern

# Background jobs
command &                  # Run in background
nohup command &            # Run detached (survives logout)
jobs                       # List background jobs
fg %1                      # Bring job 1 to foreground
```

**Understanding signals**:
```bash
# SIGTERM (15) - Polite kill, allows cleanup
kill -15 $PID

# SIGKILL (9) - Force kill, no cleanup
kill -9 $PID

# SIGHUP (1) - Reload configuration
kill -1 $(pgrep nginx)     # Reload nginx config
```

---

## 🌐 Networking Commands (Cloud Era)

### Modern Networking Tools

```bash
# IP address (replaces ifconfig)
ip addr show               # Show all interfaces
ip addr show eth0          # Show specific interface
ip route show              # Show routing table

# Socket statistics (replaces netstat)
ss -tulpn                  # All listening TCP/UDP ports with process names
ss -t state established    # Show established TCP connections

# DNS debugging
dig google.com             # DNS lookup
dig +trace google.com      # Trace DNS resolution
dig @8.8.8.8 google.com    # Query specific nameserver

# Network testing
ping -c 4 google.com       # Send 4 packets
curl -I https://google.com # HTTP headers
wget https://example.com   # Download file
nc -zv host 80             # Test if port is open
```

**Real-world troubleshooting**:
```bash
# Check if nginx is listening on port 80
ss -tulpn | grep :80

# Test connectivity to database
nc -zv db.example.com 5432

# Check DNS resolution
dig +short myapp.example.com
```

### Packet Analysis (Advanced Debugging)

```bash
# Capture traffic on port 80
tcpdump -i eth0 port 80

# Save to file for Wireshark analysis
tcpdump -i eth0 -w capture.pcap

# Filter by host
tcpdump -i eth0 host 192.168.1.100
```

---

## 🚀 Performance Analysis & Troubleshooting

### CPU Analysis

```bash
# Load average
uptime
# Output: 15:30:01 up 10 days, load average: 2.5, 1.8, 1.2
#                                             1min 5min 15min

# Rule of thumb: Load > Number of CPU cores = overloaded
nproc                      # Number of CPU cores
```

**Interpreting load average**:
- Load = 1.0 on a 4-core system: 25% utilized
- Load = 4.0 on a 4-core system: 100% utilized
- Load = 8.0 on a 4-core system: **Overloaded!**

### Memory Analysis

```bash
# Memory usage
free -h
#               total        used        free      shared  buff/cache   available
# Mem:           15Gi       5.0Gi       2.0Gi       100Mi       8.0Gi       10Gi

# Important: Look at "available", not "free"
# Linux uses free RAM for caching (buff/cache) - this is GOOD!
```

**Understanding memory**:
- **used**: Actually used by applications
- **buff/cache**: Used for disk caching (can be freed if needed)
- **available**: Memory available for new applications

### Disk I/O Analysis

```bash
# Disk usage
df -h                      # Disk space
du -sh /var/log/*          # Directory sizes

# I/O statistics
iostat -xz 1               # Every 1 second
# Look at %util column:
# - Near 100% = disk is bottleneck
# - High await = slow disk

# Which process is eating I/O?
iotop                      # Install: apt install iotop
```

**Real-world scenario: "Disk is full"**
```bash
# 1. Check usage
df -h

# 2. Find large files
du -ah / | sort -rh | head -n 20

# 3. Check for deleted files held by processes
lsof +L1
# (Common issue: log files deleted but process still writing)

# 4. Clean up
journalctl --vacuum-time=7d    # Keep only 7 days of logs
```

---

## 🔧 System Administration

### Service Management (systemd)

```bash
# Service status
systemctl status nginx

# Start/stop/restart
systemctl start nginx
systemctl stop nginx
systemctl restart nginx
systemctl reload nginx         # Reload config without restart

# Enable/disable on boot
systemctl enable nginx
systemctl disable nginx

# View logs
journalctl -u nginx            # Service logs
journalctl -u nginx -f         # Follow logs
journalctl -u nginx --since "1 hour ago"
journalctl -u nginx --since "2024-01-01"
```

**Create a simple systemd service**:
```bash
# /etc/systemd/system/myapp.service
[Unit]
Description=My Application
After=network.target

[Service]
Type=simple
User=myapp
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/python3 /opt/myapp/app.py
Restart=always

[Install]
WantedBy=multi-user.target

# Reload systemd and start
systemctl daemon-reload
systemctl start myapp
systemctl enable myapp
```

### Package Management

```bash
# Ubuntu/Debian (apt)
apt update                     # Update package list
apt upgrade                    # Upgrade packages
apt install nginx              # Install package
apt remove nginx               # Remove package
apt search keyword             # Search packages

# RHEL/CentOS (yum/dnf)
yum update                     # Update packages
yum install nginx              # Install package
yum remove nginx               # Remove package
```

---

## 🔒 Security Essentials

### File Permissions

```bash
# Understanding permissions
# rwx rwx rwx
# │   │   └── Others
# │   └────── Group
# └────────── Owner
# r=4, w=2, x=1

chmod 755 script.sh            # rwxr-xr-x
chmod 600 id_rsa               # rw------- (SSH keys MUST be 600)
chmod +x script.sh             # Add execute permission

# Ownership
chown user:group file
chown -R user:group directory
```

**Real-world example**:
```bash
# SSH key permissions (common issue)
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
chmod 644 ~/.ssh/authorized_keys
```

### SSH Hardening

```bash
# /etc/ssh/sshd_config
PermitRootLogin no             # Disable root login
PasswordAuthentication no      # Use keys only
AllowUsers user1 user2         # Whitelist users
Port 2222                      # Change default port (optional)

# Restart SSH
systemctl restart sshd
```

### Firewall (ufw)

```bash
# Enable firewall
ufw enable

# Allow services
ufw allow 22/tcp               # SSH
ufw allow 80/tcp               # HTTP
ufw allow 443/tcp              # HTTPS

# Allow from specific IP
ufw allow from 192.168.1.0/24

# Check status
ufw status

# Delete rule
ufw delete allow 80/tcp
```

---

## 🎯 Real-World Troubleshooting Scenarios

### Scenario 1: "Server is Slow"

```bash
# 1. Check load
uptime

# 2. Check CPU/Memory
htop

# 3. Check disk I/O
iostat -xz 1

# 4. Check network
iftop                          # Install: apt install iftop

# 5. Check processes
ps aux --sort=-%cpu | head     # Top CPU consumers
ps aux --sort=-%mem | head     # Top memory consumers
```

### Scenario 2: "Can't Connect to Service"

```bash
# 1. Is the service running?
systemctl status nginx

# 2. Is it listening on the port?
ss -tulpn | grep :80

# 3. Is the firewall blocking it?
ufw status

# 4. Can we connect locally?
curl http://localhost

# 5. Can we connect remotely?
curl http://server-ip

# 6. Check logs
journalctl -u nginx -n 50
```

### Scenario 3: "Disk is Full"

```bash
# 1. Check usage
df -h

# 2. Find large directories
du -sh /* | sort -rh | head -n 10

# 3. Find large files
find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null

# 4. Check for deleted files held by processes
lsof +L1

# 5. Clean up
# - Rotate logs
# - Delete old backups
# - Clean package cache: apt clean
```

---

## 🐚 Shell Scripting Basics

### Essential Script Template

```bash
#!/bin/bash
set -euo pipefail              # Exit on error, undefined var, pipe failure

# Variables
LOG_FILE="/var/log/myapp.log"
DATE=$(date +%Y-%m-%d)

# Functions
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

error_exit() {
    log "ERROR: $1"
    exit 1
}

# Main script
log "Starting backup"

if [ ! -d "/backup" ]; then
    error_exit "Backup directory does not exist"
fi

tar -czf "/backup/backup-$DATE.tar.gz" /var/www || error_exit "Backup failed"

log "Backup completed successfully"
```

### Useful Patterns

```bash
# Check if file exists
if [ -f "/path/to/file" ]; then
    echo "File exists"
fi

# Check if directory exists
if [ -d "/path/to/dir" ]; then
    echo "Directory exists"
fi

# Loop through files
for file in *.log; do
    echo "Processing $file"
done

# Read file line by line
while IFS= read -r line; do
    echo "$line"
done < file.txt
```

---

## ✅ Mastery Checklist

After mastering this guide, you should be able to:

- [ ] Navigate the file system confidently
- [ ] Understand the boot process and troubleshoot boot failures
- [ ] Use /proc and /sys for system information
- [ ] Master essential commands (find, grep, awk, sed)
- [ ] Manage processes and understand signals
- [ ] Debug network issues with modern tools (ss, dig, tcpdump)
- [ ] Analyze system performance (CPU, memory, disk I/O)
- [ ] Manage services with systemd
- [ ] Secure systems (permissions, SSH, firewall)
- [ ] Troubleshoot common production issues
- [ ] Write basic shell scripts for automation

---

## 📚 Practice Exercises

### Exercise 1: Log Analysis
```bash
# Find all ERROR lines in the last hour
journalctl --since "1 hour ago" | grep ERROR

# Count unique IPs in access log
awk '{print $1}' /var/log/nginx/access.log | sort | uniq | wc -l

# Find top 10 IPs by request count
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -10
```

### Exercise 2: System Health Check Script
```bash
#!/bin/bash
echo "=== System Health Check ==="
echo "Uptime: $(uptime)"
echo "Load: $(cat /proc/loadavg)"
echo "Memory: $(free -h | grep Mem)"
echo "Disk: $(df -h / | tail -1)"
echo "Failed Services: $(systemctl --failed --no-pager)"
```

---

**Next Step**: Networking is the nervous system of DevOps. Master it in **[Networking Fundamentals](./networking.md)**.

> **Remember**: Linux proficiency comes from daily practice. Use the command line for everything. Break things in a VM. Read man pages. The more you use it, the more natural it becomes.
