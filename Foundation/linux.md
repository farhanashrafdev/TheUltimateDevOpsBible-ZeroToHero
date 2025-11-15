# Linux Fundamentals for DevOps

## 🎯 Introduction

Linux is the foundation of modern DevOps. Most servers, containers, and cloud infrastructure run on Linux. This guide covers essential Linux skills every DevOps engineer must master.

## 📚 Core Concepts

### File System Hierarchy

```
/                    Root directory
├── /bin             Essential binaries
├── /boot            Boot files
├── /dev             Device files
├── /etc             Configuration files
├── /home            User home directories
├── /lib             Shared libraries
├── /opt             Optional software
├── /proc            Process information
├── /root            Root user home
├── /sbin            System binaries
├── /tmp             Temporary files
├── /usr             User programs
└── /var             Variable data (logs, etc.)
```

### Essential Commands

#### File Operations

```bash
# List files
ls -lah                    # Detailed list with hidden files
ls -lht                    # Sorted by time
ls -lhS                    # Sorted by size

# Change directory
cd /path/to/directory      # Absolute path
cd ..                      # Parent directory
cd ~                       # Home directory
cd -                       # Previous directory

# Create directories
mkdir -p /path/to/create   # Create with parents
mkdir dir1 dir2 dir3      # Create multiple

# Copy files
cp source dest             # Copy file
cp -r source dest         # Copy directory recursively
cp -p source dest         # Preserve attributes

# Move/rename
mv oldname newname         # Rename
mv file /path/to/         # Move

# Remove
rm file                    # Remove file
rm -rf directory          # Remove directory (careful!)
rm -i file                # Interactive removal

# Find files
find /path -name "*.log"   # Find by name
find /path -type f         # Find files only
find /path -mtime -7       # Modified in last 7 days
find /path -size +100M     # Larger than 100MB

# Search in files
grep "pattern" file        # Search in file
grep -r "pattern" /path   # Recursive search
grep -i "pattern" file    # Case insensitive
grep -v "pattern" file    # Invert match
```

#### Text Processing

```bash
# View files
cat file                   # Display entire file
less file                  # Page through file
head -n 20 file           # First 20 lines
tail -n 20 file           # Last 20 lines
tail -f file              # Follow (watch) file

# Text manipulation
cut -d: -f1 file          # Cut by delimiter
awk '{print $1}' file     # Print first field
sed 's/old/new/g' file    # Replace text
sort file                 # Sort lines
uniq file                # Remove duplicates
wc -l file               # Count lines

# Advanced text processing
# Extract IP addresses from logs
grep -oE '\b([0-9]{1,3}\.){3}[0-9]{1,3}\b' access.log

# Count unique IPs
grep -oE '\b([0-9]{1,3}\.){3}[0-9]{1,3}\b' access.log | sort | uniq | wc -l

# Find largest files
find /var/log -type f -exec ls -lh {} \; | awk '{print $5, $9}' | sort -hr | head -10
```

#### Process Management

```bash
# View processes
ps aux                     # All processes
ps aux | grep nginx       # Filter processes
top                       # Interactive process viewer
htop                      # Better top (install separately)

# Process control
kill PID                   # Kill process
kill -9 PID               # Force kill
killall process_name      # Kill by name
pkill -f pattern          # Kill by pattern

# Background jobs
command &                 # Run in background
nohup command &          # Run detached
jobs                      # List jobs
fg %1                     # Bring to foreground
bg %1                     # Send to background

# System information
uptime                    # System uptime
free -h                   # Memory usage
df -h                     # Disk usage
du -sh /path              # Directory size
```

#### Network Operations

```bash
# Network configuration
ip addr show              # Show IP addresses
ip route show             # Show routing table
ifconfig                  # Network interfaces (deprecated)
netstat -tulpn            # Network connections
ss -tulpn                 # Modern netstat

# Network testing
ping host                 # Ping host
traceroute host           # Trace route
curl -I url               # HTTP headers
wget url                  # Download file

# Port checking
nc -zv host port          # Test port
telnet host port          # Test connection
```

#### System Administration

```bash
# User management
whoami                    # Current user
id                        # User and group IDs
sudo command              # Run as root
su -                      # Switch to root

# Package management (Ubuntu/Debian)
apt update                # Update package list
apt upgrade               # Upgrade packages
apt install package      # Install package
apt remove package       # Remove package
apt search keyword       # Search packages

# Package management (RHEL/CentOS)
yum update                # Update packages
yum install package      # Install package
yum remove package       # Remove package

# Service management (systemd)
systemctl status service  # Service status
systemctl start service   # Start service
systemctl stop service    # Stop service
systemctl restart service # Restart service
systemctl enable service  # Enable on boot
systemctl disable service # Disable on boot

# Logs
journalctl -u service     # Service logs
journalctl -f             # Follow logs
journalctl --since "1 hour ago"
dmesg                     # Kernel messages
```

## 🔧 Advanced Topics

### Permissions

```bash
# Understanding permissions
# rwx rwx rwx
# owner group others
# r=4, w=2, x=1

chmod 755 file            # rwxr-xr-x
chmod +x file             # Add execute
chmod -w file             # Remove write
chmod u+x file            # User execute
chmod g+w file            # Group write
chmod o-r file            # Others read

# Ownership
chown user:group file     # Change owner
chown -R user:group dir   # Recursive

# Special permissions
chmod +s file             # Setuid/setgid
chmod +t dir              # Sticky bit
```

### File System Operations

```bash
# Disk operations
fdisk -l                  # List partitions
lsblk                     # Block devices
mount /dev/sda1 /mnt      # Mount filesystem
umount /mnt               # Unmount
df -h                     # Disk space
du -h --max-depth=1       # Directory sizes

# File system types
ext4                      # Standard Linux
xfs                       # High performance
btrfs                     # Advanced features
```

### Environment Variables

```bash
# View variables
env                       # All environment variables
echo $PATH                # Specific variable
printenv VAR              # Print variable

# Set variables
export VAR=value          # Export to environment
VAR=value command         # Set for command only

# Common variables
$HOME                     # Home directory
$USER                     # Username
$PATH                     # Command search path
$PWD                      # Current directory
$SHELL                    # Shell program
```

### Shell Scripting Basics

```bash
#!/bin/bash
# Simple script example

# Variables
NAME="DevOps"
VERSION=1.0

# Output
echo "Hello, $NAME!"
echo "Version: $VERSION"

# Conditionals
if [ -f "$1" ]; then
    echo "File exists"
else
    echo "File not found"
fi

# Loops
for file in *.log; do
    echo "Processing $file"
done

# Functions
function cleanup() {
    echo "Cleaning up..."
    rm -f /tmp/temp*
}
```

### System Monitoring

```bash
# CPU monitoring
top                       # CPU usage
htop                      # Better interface
vmstat 1                 # Virtual memory stats
iostat 1                 # I/O statistics

# Memory monitoring
free -h                   # Memory usage
cat /proc/meminfo         # Detailed memory info

# Disk I/O
iostat -x 1               # Disk I/O stats
iotop                     # I/O by process

# Network monitoring
iftop                     # Network traffic
nethogs                   # Network by process
```

## 🐳 Docker & Container Commands

```bash
# Container operations
docker ps                  # Running containers
docker ps -a              # All containers
docker exec -it container bash  # Execute in container
docker logs container     # Container logs
docker stats              # Container stats

# Image operations
docker images             # List images
docker pull image         # Pull image
docker build -t image .   # Build image
docker rmi image          # Remove image
```

## 🔐 Security Best Practices

```bash
# SSH keys
ssh-keygen -t rsa -b 4096  # Generate key
ssh-copy-id user@host     # Copy key to host

# Firewall (ufw)
ufw enable                # Enable firewall
ufw allow 22/tcp          # Allow SSH
ufw status                # Check status

# File integrity
md5sum file               # MD5 checksum
sha256sum file            # SHA256 checksum

# Secure file transfer
scp file user@host:/path  # Secure copy
rsync -avz source dest    # Synchronize
```

## 📝 Common DevOps Tasks

### Log Analysis

```bash
# Find errors in logs
grep -i error /var/log/app.log
grep -E "ERROR|WARN" /var/log/app.log

# Count occurrences
grep "pattern" file | wc -l

# Extract timestamps
grep -oE '[0-9]{4}-[0-9]{2}-[0-9]{2} [0-9]{2}:[0-9]{2}:[0-9]{2}' logfile

# Top IP addresses
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10
```

### Disk Cleanup

```bash
# Find large files
find / -type f -size +100M 2>/dev/null

# Clean package cache
apt clean                 # Debian/Ubuntu
yum clean all            # RHEL/CentOS

# Clean logs
journalctl --vacuum-time=7d  # Keep 7 days
find /var/log -type f -mtime +30 -delete
```

### System Maintenance

```bash
# Update system
apt update && apt upgrade -y  # Debian/Ubuntu
yum update -y              # RHEL/CentOS

# Check disk space
df -h
du -sh /* | sort -h

# Check memory
free -h
cat /proc/meminfo | grep MemAvailable

# Check load
uptime
cat /proc/loadavg
```

## 🎯 Practice Exercises

### Exercise 1: Log Analysis
```bash
# Create sample log file
cat > access.log << EOF
192.168.1.1 - - [25/Oct/2024:10:15:30] "GET /page1 HTTP/1.1" 200
192.168.1.2 - - [25/Oct/2024:10:15:31] "GET /page2 HTTP/1.1" 404
192.168.1.1 - - [25/Oct/2024:10:15:32] "GET /page1 HTTP/1.1" 200
EOF

# Tasks:
# 1. Count total requests
wc -l access.log

# 2. Find unique IPs
awk '{print $1}' access.log | sort | uniq

# 3. Count requests per IP
awk '{print $1}' access.log | sort | uniq -c

# 4. Find 404 errors
grep "404" access.log
```

### Exercise 2: System Monitoring Script
```bash
#!/bin/bash
# system_check.sh

echo "=== System Health Check ==="
echo "Uptime: $(uptime)"
echo "Memory: $(free -h | grep Mem)"
echo "Disk: $(df -h / | tail -1)"
echo "Load: $(cat /proc/loadavg)"
```

### Exercise 3: Backup Script
```bash
#!/bin/bash
# backup.sh

SOURCE="/var/www"
DEST="/backup/$(date +%Y%m%d)"
mkdir -p "$DEST"
tar -czf "$DEST/backup.tar.gz" "$SOURCE"
```

## 📚 Cheat Sheet

```bash
# Quick reference
Ctrl+C          # Cancel command
Ctrl+D          # Exit shell
Ctrl+Z          # Suspend process
Ctrl+L          # Clear screen
!!              # Last command
!$              # Last argument
history         # Command history
alias ll='ls -lah'  # Create alias

# Redirection
command > file      # Redirect output
command >> file     # Append output
command < file      # Redirect input
command 2> file     # Redirect errors
command &> file     # Redirect all

# Pipes
command1 | command2  # Pipe output
command1 | tee file  # Output and save
```

## ✅ Mastery Checklist

- [ ] Navigate file system confidently
- [ ] Manage files and directories
- [ ] Understand permissions
- [ ] Use text processing tools
- [ ] Manage processes
- [ ] Configure networking
- [ ] Write basic shell scripts
- [ ] Monitor system resources
- [ ] Analyze logs effectively
- [ ] Perform system administration tasks

---

**Remember**: Linux proficiency is fundamental to DevOps. Practice daily, build scripts, and automate tasks. Mastery comes through hands-on experience.

