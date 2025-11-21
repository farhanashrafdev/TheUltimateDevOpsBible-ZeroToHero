# Lab: Linux Troubleshooting Challenge

## 🎯 Scenario
You are the on-call engineer. A developer reports: "The web server is down!" You have SSH access. Fix it.

## 🛠️ Setup
Run this script to break your system (in a VM/Container!):
```bash
# break_system.sh
# 1. Fill the disk
dd if=/dev/zero of=/var/log/bigfile bs=1M count=10000
# 2. Stop the service
systemctl stop nginx
# 3. Change permissions
chmod 000 /var/www/html/index.html
# 4. Block port
iptables -A INPUT -p tcp --dport 80 -j DROP
```

## 🕵️ Investigation Steps

### 1. Check Service Status
```bash
systemctl status nginx
# Output: Inactive (dead)
```
**Action**: `systemctl start nginx`
**Result**: Fails. "No space left on device".

### 2. Check Disk Space
```bash
df -h
# Output: /var is 100% full
```
**Action**: Find large files.
```bash
find /var -type f -size +100M
rm /var/log/bigfile
```
**Action**: Start nginx. `systemctl start nginx`. Success?

### 3. Check Connectivity
**Action**: `curl localhost`
**Result**: "403 Forbidden".

### 4. Check Permissions
**Action**: Check logs `tail /var/log/nginx/error.log`
**Result**: "Permission denied".
**Action**: Fix permissions.
```bash
chmod 644 /var/www/html/index.html
```

### 5. Check Network
**Action**: `curl localhost` works. But external access fails.
**Action**: Check firewall.
```bash
iptables -L
```
**Result**: DROP rule on port 80.
**Action**: Delete rule.
```bash
iptables -D INPUT -p tcp --dport 80 -j DROP
```

## ✅ Success Criteria
- [ ] Nginx is running.
- [ ] Disk has free space.
- [ ] `curl localhost` returns 200.
- [ ] External IP returns 200.
