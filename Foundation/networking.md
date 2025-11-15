# Networking Fundamentals for DevOps

## 🎯 Introduction

Understanding networking is crucial for DevOps engineers. You'll work with load balancers, service meshes, Kubernetes networking, cloud networking, and troubleshooting connectivity issues. This guide covers essential networking concepts.

## 📚 OSI Model

### 7-Layer Model

```
Layer 7: Application    (HTTP, HTTPS, DNS, SSH, FTP)
Layer 6: Presentation   (SSL/TLS, encryption)
Layer 5: Session        (Sockets, sessions)
Layer 4: Transport      (TCP, UDP)
Layer 3: Network        (IP, ICMP, routing)
Layer 2: Data Link     (Ethernet, MAC addresses)
Layer 1: Physical      (Cables, signals)
```

### TCP/IP Model (Simplified)

```
Application Layer    (HTTP, DNS, SSH)
Transport Layer      (TCP, UDP)
Internet Layer       (IP, ICMP)
Network Interface     (Ethernet)
```

## 🌐 IP Addressing

### IPv4

**Address Format**: `192.168.1.1` (32 bits, 4 octets)

**Private Ranges**:
- `10.0.0.0/8` (10.0.0.0 - 10.255.255.255)
- `172.16.0.0/12` (172.16.0.0 - 172.31.255.255)
- `192.168.0.0/16` (192.168.0.0 - 192.168.255.255)

**CIDR Notation**:
```
192.168.1.0/24
├── Network: 192.168.1.0
├── Subnet Mask: 255.255.255.0
├── Usable IPs: 192.168.1.1 - 192.168.1.254
└── Broadcast: 192.168.1.255
```

### IPv6

**Address Format**: `2001:0db8:85a3:0000:0000:8a2e:0370:7334`

**Shortened**: `2001:db8:85a3::8a2e:370:7334`

## 🔌 TCP vs UDP

### TCP (Transmission Control Protocol)
- **Connection-oriented**: Establishes connection first
- **Reliable**: Guarantees delivery
- **Ordered**: Maintains order
- **Flow control**: Manages data rate
- **Use cases**: HTTP, HTTPS, SSH, FTP, email

**TCP Three-Way Handshake**:
```
Client                Server
  │                     │
  │─── SYN ────────────▶│
  │                     │
  │◀── SYN-ACK ─────────│
  │                     │
  │─── ACK ────────────▶│
  │                     │
  │◀── Data ────────────│
```

### UDP (User Datagram Protocol)
- **Connectionless**: No connection needed
- **Unreliable**: No delivery guarantee
- **Fast**: Lower overhead
- **Use cases**: DNS, DHCP, video streaming, gaming

## 🌍 DNS (Domain Name System)

### DNS Resolution Process

```
1. Check local cache
2. Check /etc/hosts
3. Query local DNS server
4. Query root DNS servers
5. Query TLD servers (.com)
6. Query authoritative servers
7. Return IP address
```

### Common DNS Records

```bash
# A Record (IPv4)
example.com.    A    192.168.1.1

# AAAA Record (IPv6)
example.com.    AAAA 2001:db8::1

# CNAME (Canonical Name)
www.example.com. CNAME example.com.

# MX (Mail Exchange)
example.com.    MX   10 mail.example.com.

# TXT (Text)
example.com.    TXT  "v=spf1 ..."
```

### DNS Commands

```bash
# Query DNS
nslookup example.com
dig example.com
dig @8.8.8.8 example.com  # Use specific DNS server
dig -x 192.168.1.1        # Reverse lookup

# Check DNS resolution
host example.com
getent hosts example.com
```

## 🔐 HTTP/HTTPS

### HTTP Methods

```
GET     - Retrieve resource
POST   - Create resource
PUT    - Update resource
PATCH  - Partial update
DELETE - Delete resource
HEAD   - Get headers only
OPTIONS - Get allowed methods
```

### HTTP Status Codes

```
2xx Success
  200 OK
  201 Created
  204 No Content

3xx Redirection
  301 Moved Permanently
  302 Found
  304 Not Modified

4xx Client Error
  400 Bad Request
  401 Unauthorized
  403 Forbidden
  404 Not Found
  429 Too Many Requests

5xx Server Error
  500 Internal Server Error
  502 Bad Gateway
  503 Service Unavailable
  504 Gateway Timeout
```

### HTTPS/TLS

**TLS Handshake**:
```
Client                Server
  │                     │
  │─── ClientHello ────▶│
  │                     │
  │◀── ServerHello ─────│
  │◀── Certificate ─────│
  │                     │
  │─── ClientKeyExchange│
  │                     │
  │◀── Finished ────────│
  │                     │
  │─── Encrypted Data ──▶│
```

## 🔧 Network Tools

### Basic Tools

```bash
# Ping
ping -c 4 google.com      # 4 packets
ping -i 2 google.com       # 2 second interval

# Traceroute
traceroute google.com
tracepath google.com

# Network configuration
ip addr show               # Show IP addresses
ip link show               # Show interfaces
ip route show              # Show routing table

# Network connections
netstat -tulpn             # All connections
ss -tulpn                  # Modern netstat
lsof -i :80                # Processes using port 80

# Network testing
curl -I https://example.com  # HTTP headers
wget https://example.com     # Download
nc -zv host 80              # Test port
telnet host 80              # Test connection
```

### Advanced Tools

```bash
# tcpdump (Packet capture)
tcpdump -i eth0
tcpdump -i eth0 port 80
tcpdump -i eth0 -w capture.pcap

# wireshark (GUI packet analyzer)
# Install: apt install wireshark
wireshark

# nmap (Network scanner)
nmap 192.168.1.1
nmap -p 80,443 example.com
nmap -sP 192.168.1.0/24    # Ping scan

# iptables (Firewall)
iptables -L                # List rules
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -j DROP
```

## 🌐 Load Balancing

### Types of Load Balancing

**Layer 4 (Transport)**:
- Based on IP and port
- Fast, simple
- No content awareness

**Layer 7 (Application)**:
- Based on HTTP content
- More intelligent
- Can do SSL termination

### Load Balancing Algorithms

```
Round Robin        - Equal distribution
Least Connections  - Fewest active connections
IP Hash           - Based on client IP
Weighted          - Based on server capacity
```

### Example: Nginx Load Balancer

```nginx
upstream backend {
    least_conn;
    server 192.168.1.10:8080 weight=3;
    server 192.168.1.11:8080 weight=2;
    server 192.168.1.12:8080 backup;
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

## 🔒 Firewall Concepts

### Common Ports

```
20, 21    - FTP
22        - SSH
23        - Telnet
25        - SMTP
53        - DNS
80        - HTTP
443       - HTTPS
3306      - MySQL
5432      - PostgreSQL
6379      - Redis
8080      - HTTP (alternative)
8443      - HTTPS (alternative)
```

### Firewall Rules (iptables)

```bash
# Allow SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow HTTP/HTTPS
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow from specific IP
iptables -A INPUT -s 192.168.1.0/24 -j ACCEPT

# Drop all other traffic
iptables -A INPUT -j DROP

# Save rules
iptables-save > /etc/iptables/rules.v4
```

## ☁️ Cloud Networking

### AWS Networking

**VPC (Virtual Private Cloud)**:
```
VPC (10.0.0.0/16)
├── Public Subnet (10.0.1.0/24)
│   └── Internet Gateway
├── Private Subnet (10.0.2.0/24)
│   └── NAT Gateway
└── Database Subnet (10.0.3.0/24)
```

**Key Components**:
- **VPC**: Isolated network
- **Subnet**: Network segment
- **Internet Gateway**: Internet access
- **NAT Gateway**: Outbound internet for private subnets
- **Security Groups**: Firewall rules
- **Route Tables**: Routing rules

### GCP Networking

**VPC Network**:
- Global VPCs
- Subnets (regional)
- Firewall rules
- Cloud Load Balancing

### Azure Networking

**Virtual Network**:
- VNets
- Subnets
- Network Security Groups
- Application Gateway

## 🐳 Container Networking

### Docker Networking

```bash
# Network types
docker network ls

# Create network
docker network create mynetwork

# Bridge network (default)
docker run --network bridge nginx

# Host network
docker run --network host nginx

# None network (isolated)
docker run --network none nginx

# Custom network
docker network create --driver bridge --subnet=172.20.0.0/16 mynet
```

### Docker Network Drivers

```
bridge    - Default, isolated network
host      - Use host network
overlay   - Multi-host networking
macvlan   - MAC address per container
none      - No networking
```

## ☸️ Kubernetes Networking

### Pod Networking

```
Pod Network Model:
- Every pod gets unique IP
- Pods can communicate directly
- No NAT between pods
```

### Services

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: ClusterIP  # or NodePort, LoadBalancer
```

### Service Types

```
ClusterIP    - Internal only
NodePort     - External via node IP
LoadBalancer - External via cloud LB
ExternalName - External service
```

### Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
    - host: example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-service
                port:
                  number: 80
```

## 🔍 Network Troubleshooting

### Common Issues

**1. Cannot reach host**
```bash
# Check connectivity
ping host
traceroute host

# Check DNS
nslookup host
dig host

# Check firewall
iptables -L
ufw status
```

**2. Port not accessible**
```bash
# Check if service is listening
ss -tulpn | grep :80
netstat -tulpn | grep :80

# Check firewall
iptables -L -n | grep 80
ufw status | grep 80

# Test from remote
nc -zv host 80
telnet host 80
```

**3. Slow network**
```bash
# Check bandwidth
iperf3 -c server
speedtest-cli

# Check latency
ping -c 10 host
mtr host

# Check packet loss
ping -c 100 host | grep loss
```

**4. DNS issues**
```bash
# Test DNS resolution
dig @8.8.8.8 example.com
nslookup example.com

# Check /etc/resolv.conf
cat /etc/resolv.conf

# Flush DNS cache
systemd-resolve --flush-caches
```

## 📊 Network Monitoring

### Tools

```bash
# iftop (interface top)
iftop -i eth0

# nethogs (network by process)
nethogs

# vnstat (network statistics)
vnstat -i eth0

# iptraf (network monitor)
iptraf-ng

# tcpdump (packet capture)
tcpdump -i eth0 -w capture.pcap
```

### Monitoring Metrics

```
Bandwidth    - Data transfer rate
Latency      - Round-trip time
Packet Loss  - Dropped packets
Throughput  - Actual data rate
Jitter       - Latency variation
```

## 🎯 Practice Exercises

### Exercise 1: Network Configuration

```bash
# Configure static IP (Ubuntu)
sudo nano /etc/netplan/01-netcfg.yaml

network:
  version: 2
  ethernets:
    eth0:
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4

sudo netplan apply
```

### Exercise 2: Port Forwarding

```bash
# SSH port forwarding
ssh -L 8080:localhost:80 user@remote-host

# Access localhost:8080 → remote:80
```

### Exercise 3: Network Testing Script

```bash
#!/bin/bash
# network_test.sh

HOST="google.com"
PORT=80

echo "Testing $HOST:$PORT"
if nc -zv $HOST $PORT 2>&1 | grep -q succeeded; then
    echo "✓ Connection successful"
else
    echo "✗ Connection failed"
fi
```

## 📚 Key Concepts Summary

### Subnetting

```
192.168.1.0/24
├── Network: 192.168.1.0
├── Subnet Mask: 255.255.255.0
├── Usable IPs: 254 (192.168.1.1 - 192.168.1.254)
└── Broadcast: 192.168.1.255

192.168.1.0/26
├── Network: 192.168.1.0
├── Subnet Mask: 255.255.255.192
├── Usable IPs: 62 (192.168.1.1 - 192.168.1.62)
└── Broadcast: 192.168.1.63
```

### Routing

```bash
# View routing table
ip route show
route -n

# Add route
ip route add 192.168.2.0/24 via 192.168.1.1

# Default gateway
ip route add default via 192.168.1.1
```

## ✅ Mastery Checklist

- [ ] Understand OSI/TCP-IP models
- [ ] Configure IP addresses and subnets
- [ ] Understand TCP vs UDP
- [ ] Configure DNS
- [ ] Understand HTTP/HTTPS
- [ ] Use network troubleshooting tools
- [ ] Configure firewalls
- [ ] Understand load balancing
- [ ] Configure container networking
- [ ] Understand Kubernetes networking
- [ ] Troubleshoot network issues
- [ ] Monitor network performance

---

**Remember**: Networking is fundamental to DevOps. Practice with real networks, understand cloud networking, and master troubleshooting. Hands-on experience is essential.

