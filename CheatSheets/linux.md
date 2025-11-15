# Linux Cheat Sheet

## 📚 Essential Commands

### File Operations
```bash
ls -lah              # List with details
find /path -name "*.log"  # Find files
grep "pattern" file  # Search in file
cat file             # Display file
head -n 20 file      # First 20 lines
tail -f file         # Follow file
```

### Process Management
```bash
ps aux               # All processes
top                  # Process viewer
kill PID             # Kill process
killall process      # Kill by name
jobs                 # Background jobs
```

### System Info
```bash
df -h                # Disk usage
du -sh /path         # Directory size
free -h              # Memory
uptime               # System info
```

### Networking
```bash
ip addr show         # IP addresses
netstat -tulpn       # Network connections
ping host            # Test connectivity
curl -I url          # HTTP headers
```

### Permissions
```bash
chmod 755 file       # Set permissions
chown user:group file # Change owner
chmod +x file        # Add execute
```

---

**Quick Reference**: Essential Linux commands for DevOps.

