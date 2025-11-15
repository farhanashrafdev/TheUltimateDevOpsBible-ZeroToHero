# Linux Cheat Sheet - Complete Reference for DevOps

## 📁 File Operations

### Listing Files

```bash
# List files
ls                                # Basic list
ls -l                             # Long format
ls -la                            # All files (including hidden)
ls -lah                           # Human readable sizes
ls -lht                           # Sorted by time
ls -lhS                           # Sorted by size
ls -lR                            # Recursive
ls -d */                          # Directories only
ls *.txt                          # Pattern matching
ls -1                             # One per line
```

### Navigation

```bash
# Change directory
cd /path/to/directory             # Absolute path
cd ..                             # Parent directory
cd ~                              # Home directory
cd -                              # Previous directory
cd                                # Home directory

# Show current directory
pwd
```

### File Operations

```bash
# Create files/directories
touch file.txt                    # Create empty file
mkdir directory                    # Create directory
mkdir -p path/to/create           # Create with parents
mkdir dir1 dir2 dir3              # Multiple directories

# Copy files
cp source dest                     # Copy file
cp -r source dest                 # Copy directory
cp -p source dest                 # Preserve attributes
cp -a source dest                 # Archive mode
cp -u source dest                 # Update only
cp -v source dest                 # Verbose

# Move/rename
mv oldname newname                 # Rename
mv file /path/to/                  # Move
mv -i file dest                    # Interactive

# Remove
rm file                           # Remove file
rm -f file                        # Force
rm -r directory                   # Remove directory
rm -rf directory                  # Force recursive (careful!)
rm -i file                        # Interactive
rm -v file                        # Verbose
```

### Finding Files

```bash
# Find files
find /path -name "*.log"          # By name
find /path -type f                # Files only
find /path -type d                # Directories only
find /path -mtime -7              # Modified in last 7 days
find /path -mtime +30             # Modified more than 30 days ago
find /path -size +100M            # Larger than 100MB
find /path -size -1k              # Smaller than 1KB
find /path -user username         # By owner
find /path -perm 644              # By permissions
find /path -name "*.log" -delete  # Find and delete

# Locate (faster, requires database)
locate filename
updatedb                          # Update locate database
```

### File Content

```bash
# View files
cat file                          # Display entire file
cat file1 file2                   # Multiple files
cat -n file                       # With line numbers
tac file                          # Reverse order
less file                         # Page through file
more file                         # Page through file
head -n 20 file                   # First 20 lines
tail -n 20 file                   # Last 20 lines
tail -f file                      # Follow (watch) file
tail -F file                      # Follow with retry
```

### Text Processing

```bash
# Search in files
grep "pattern" file               # Search pattern
grep -r "pattern" /path           # Recursive
grep -i "pattern" file            # Case insensitive
grep -v "pattern" file            # Invert match
grep -n "pattern" file            # With line numbers
grep -c "pattern" file            # Count matches
grep -l "pattern" /path           # List files with matches
grep -E "pattern" file            # Extended regex
grep -A 3 "pattern" file          # 3 lines after
grep -B 3 "pattern" file          # 3 lines before
grep -C 3 "pattern" file          # 3 lines context

# Text manipulation
cut -d: -f1 file                  # Cut by delimiter
cut -c1-10 file                   # Cut by characters
awk '{print $1}' file             # Print first field
awk -F: '{print $1}' file         # Custom delimiter
sed 's/old/new/g' file            # Replace text
sed -i 's/old/new/g' file         # In-place edit
sort file                         # Sort lines
sort -r file                      # Reverse sort
sort -n file                      # Numeric sort
uniq file                         # Remove duplicates
uniq -c file                      # Count occurrences
wc -l file                        # Count lines
wc -w file                        # Count words
wc -c file                        # Count characters
```

## 🔄 Process Management

### Viewing Processes

```bash
# List processes
ps                                # Current user processes
ps aux                            # All processes
ps aux | grep nginx               # Filter processes
ps -ef                            # Full format
ps -p PID                         # Specific process
ps -u username                    # By user

# Interactive process viewer
top                               # Process viewer
htop                              # Better top (install separately)
top -p PID                        # Specific process
top -u username                    # By user
```

### Process Control

```bash
# Kill processes
kill PID                          # Kill process
kill -9 PID                       # Force kill
killall process_name              # Kill by name
pkill -f pattern                  # Kill by pattern
kill -HUP PID                     # Send HUP signal
kill -TERM PID                    # Send TERM signal

# Process priority
nice -n 10 command                # Run with lower priority
renice 10 PID                     # Change priority
```

### Background Jobs

```bash
# Background jobs
command &                         # Run in background
nohup command &                   # Run detached
jobs                              # List jobs
fg %1                             # Bring to foreground
bg %1                             # Send to background
disown %1                         # Disown job
```

## 💻 System Information

### System Status

```bash
# System info
uname -a                          # System information
uname -r                          # Kernel version
hostname                          # Hostname
uptime                            # Uptime and load
whoami                            # Current user
id                                # User and group IDs
w                                 # Who is logged in
who                               # Logged in users
last                              # Last logged in users
```

### Resource Usage

```bash
# Memory
free                              # Memory usage
free -h                           # Human readable
free -m                           # Megabytes
cat /proc/meminfo                 # Detailed memory info
vmstat 1                          # Virtual memory stats

# CPU
top                               # CPU usage
htop                              # Better interface
mpstat 1                          # CPU statistics
lscpu                             # CPU information
cat /proc/cpuinfo                 # CPU details

# Disk
df -h                             # Disk usage
df -i                             # Inode usage
du -h /path                       # Directory size
du -sh /path                      # Summary
du -h --max-depth=1 /path         # One level deep
du -h --max-depth=1 /path | sort -h  # Sorted
lsblk                             # Block devices
fdisk -l                          # Partitions
```

### Disk I/O

```bash
# I/O statistics
iostat 1                          # I/O stats
iostat -x 1                       # Extended stats
iotop                             # I/O by process
```

## 🌐 Networking

### Network Configuration

```bash
# IP addresses
ip addr show                      # Show IP addresses
ip addr add 192.168.1.100/24 dev eth0  # Add IP
ip link show                      # Show interfaces
ip route show                     # Show routing table
ifconfig                          # Network interfaces (deprecated)
hostname -I                       # IP address

# Network connections
netstat -tulpn                    # All connections
ss -tulpn                         # Modern netstat
ss -tulpn | grep :80              # Filter by port
lsof -i :80                       # Processes using port 80
lsof -i -P -n                     # All network connections
```

### Network Testing

```bash
# Connectivity
ping host                         # Ping host
ping -c 4 host                    # 4 packets
ping -i 2 host                    # 2 second interval
ping6 host                        # IPv6 ping

# Route tracing
traceroute host                   # Trace route
tracepath host                    # Trace path
mtr host                         # My traceroute

# DNS
nslookup host                     # DNS lookup
dig host                          # DNS lookup
dig @8.8.8.8 host                 # Use specific DNS
dig -x 192.168.1.1                # Reverse lookup
host host                         # Host lookup
getent hosts host                 # Get host entry

# HTTP testing
curl http://example.com           # HTTP request
curl -I http://example.com        # Headers only
curl -L http://example.com        # Follow redirects
curl -O http://example.com/file   # Download file
wget http://example.com/file      # Download file
wget -r http://example.com        # Recursive download
```

### Network Monitoring

```bash
# Network traffic
iftop -i eth0                     # Interface top
nethogs                           # Network by process
vnstat -i eth0                    # Network statistics
tcpdump -i eth0                   # Packet capture
tcpdump -i eth0 port 80           # Filter by port
tcpdump -i eth0 -w capture.pcap   # Save to file
```

## 🔐 Permissions

### Understanding Permissions

```bash
# Permission format: rwx rwx rwx
#                  owner group others
# r=4, w=2, x=1

# View permissions
ls -l file                         # Long format
stat file                          # Detailed info
```

### Changing Permissions

```bash
# chmod (change mode)
chmod 755 file                     # rwxr-xr-x
chmod +x file                      # Add execute
chmod -w file                      # Remove write
chmod u+x file                     # User execute
chmod g+w file                     # Group write
chmod o-r file                     # Others read
chmod a+x file                     # All execute
chmod -R 755 directory             # Recursive

# chown (change owner)
chown user:group file              # Change owner
chown user file                    # Change user only
chown :group file                  # Change group only
chown -R user:group directory      # Recursive

# Special permissions
chmod +s file                      # Setuid/setgid
chmod +t directory                 # Sticky bit
```

## 📦 Package Management

### Debian/Ubuntu (apt)

```bash
# Update package list
apt update
apt-get update

# Upgrade packages
apt upgrade
apt-get upgrade
apt full-upgrade

# Install package
apt install package
apt-get install package
apt install package1 package2     # Multiple packages

# Remove package
apt remove package
apt purge package                 # Remove with config
apt autoremove                    # Remove unused

# Search packages
apt search keyword
apt-cache search keyword

# Package information
apt show package
apt-cache show package
dpkg -l                           # List installed
dpkg -L package                   # Files in package
```

### RHEL/CentOS (yum/dnf)

```bash
# Update packages
yum update
dnf update

# Install package
yum install package
dnf install package

# Remove package
yum remove package
dnf remove package

# Search packages
yum search keyword
dnf search keyword

# Package information
yum info package
dnf info package
rpm -qa                           # List installed
rpm -ql package                   # Files in package
```

## 🔧 System Services

### systemd

```bash
# Service management
systemctl status service           # Service status
systemctl start service           # Start service
systemctl stop service            # Stop service
systemctl restart service         # Restart service
systemctl reload service          # Reload configuration
systemctl enable service          # Enable on boot
systemctl disable service         # Disable on boot
systemctl is-enabled service      # Check if enabled
systemctl is-active service       # Check if active

# List services
systemctl list-units              # All units
systemctl list-units --type=service  # Services only
systemctl list-units --state=running  # Running only

# Logs
journalctl -u service             # Service logs
journalctl -f                     # Follow logs
journalctl --since "1 hour ago"   # Since time
journalctl --since today          # Since today
journalctl -p err                 # Errors only
```

## 📊 Monitoring

### System Monitoring

```bash
# Load average
uptime                            # Uptime and load
cat /proc/loadavg                 # Load average

# System resources
sar 1                             # System activity
sar -u 1                          # CPU
sar -r 1                          # Memory
sar -d 1                          # Disk

# Process monitoring
ps aux --sort=-%mem | head        # Top memory
ps aux --sort=-%cpu | head        # Top CPU
```

## 🔍 Logs

### Log Files

```bash
# Common log locations
/var/log/syslog                   # System log
/var/log/messages                 # Messages
/var/log/auth.log                 # Authentication
/var/log/nginx/                   # Nginx logs
/var/log/apache2/                 # Apache logs

# View logs
tail -f /var/log/syslog           # Follow log
tail -n 100 /var/log/syslog       # Last 100 lines
grep "error" /var/log/syslog      # Search in log
journalctl -f                     # Systemd logs
```

## 🛠️ Useful Commands

### Compression

```bash
# tar
tar -czf archive.tar.gz directory # Create gzip archive
tar -xzf archive.tar.gz           # Extract gzip archive
tar -cjf archive.tar.bz2 directory # Create bzip2 archive
tar -xjf archive.tar.bz2          # Extract bzip2 archive
tar -tf archive.tar.gz            # List contents

# zip
zip -r archive.zip directory      # Create zip
unzip archive.zip                 # Extract zip

# gzip
gzip file                         # Compress
gunzip file.gz                    # Decompress
```

### File Comparison

```bash
# Compare files
diff file1 file2                  # Differences
diff -u file1 file2               # Unified format
cmp file1 file2                   # Compare files
comm file1 file2                  # Common lines
```

### Environment Variables

```bash
# View variables
env                               # All environment variables
echo $PATH                        # Specific variable
printenv VAR                      # Print variable

# Set variables
export VAR=value                  # Export to environment
VAR=value command                 # Set for command only
```

## 🚨 Troubleshooting

### Common Tasks

```bash
# Find large files
find / -type f -size +100M 2>/dev/null
du -h / | sort -h | tail -20

# Find processes using port
lsof -i :80
netstat -tulpn | grep :80
ss -tulpn | grep :80

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

## 📚 Quick Reference

### Redirection

```bash
command > file                    # Redirect output
command >> file                   # Append output
command < file                    # Redirect input
command 2> file                   # Redirect errors
command &> file                   # Redirect all
command | command2                # Pipe
command | tee file                # Output and save
```

### Keyboard Shortcuts

```bash
Ctrl+C                            # Cancel command
Ctrl+D                            # Exit shell
Ctrl+Z                            # Suspend process
Ctrl+L                            # Clear screen
Ctrl+R                            # Search history
Ctrl+A                            # Beginning of line
Ctrl+E                            # End of line
Ctrl+U                            # Clear to beginning
Ctrl+K                            # Clear to end
```

---

**Remember**: Linux commands are powerful. Always verify before executing destructive operations. Use `man command` for help.
