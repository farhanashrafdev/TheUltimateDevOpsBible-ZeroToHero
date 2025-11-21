# Networking Fundamentals for DevOps: Cloud Era Networking

> **"It's always DNS."** - The Sysadmin Mantra

## 🎯 Introduction

In the cloud era, networking is **software-defined**. You're not plugging in cables; you're defining Virtual Private Clouds (VPCs), configuring Service Meshes, and debugging latency across microservices spread across multiple regions.

This guide focuses on **practical networking knowledge for DevOps engineers**.

### What You'll Learn
- Cloud networking (VPC, subnets, routing)
- Container networking (Docker, Kubernetes, CNI)
- Service Mesh concepts
- DNS and TLS deep dive
- Network troubleshooting

### What This Is NOT
- Not CCNA certification prep
- Not low-level TCP/IP protocol details
- Not physical network cabling

---

## 📚 The OSI Model (DevOps Edition)

You only really need to care about **Layers 3, 4, and 7**.

| Layer | Protocol | DevOps Relevance | Tools |
|:---|:---|:---|:---|
| **L7 Application** | HTTP, gRPC, DNS | Load Balancers (ALB), API Gateways, WAFs | curl, dig |
| **L4 Transport** | TCP, UDP | Network Load Balancers (NLB), Firewalls | ss, netstat |
| **L3 Network** | IP, ICMP | Routing, VPC Peering, Subnets, NAT | ip, ping, traceroute |
| L2 Data Link | Ethernet, MAC | (Mostly abstracted in cloud) | - |
| L1 Physical | Cables, signals | (Cloud provider's problem) | - |

**Real-world example**:
```
User request → L7 (ALB routes based on /api path) 
            → L4 (NLB distributes TCP connections)
            → L3 (Routes to correct subnet/AZ)
            → Your application
```

---

## ☁️ Cloud Networking (AWS/GCP/Azure)

### 1. VPC (Virtual Private Cloud)

Your isolated slice of the cloud.

```
VPC (10.0.0.0/16)
├── Public Subnet (10.0.1.0/24)
│   ├── Internet Gateway (IGW)
│   └── Web Servers
├── Private Subnet (10.0.2.0/24)
│   ├── NAT Gateway (for outbound)
│   └── Application Servers
└── Database Subnet (10.0.3.0/24)
    └── RDS Instances
```

**Key Concepts**:
- **CIDR Blocks**: IP address ranges
  - `/32` = 1 IP (single host)
  - `/24` = 256 IPs (standard subnet)
  - `/16` = 65,536 IPs (standard VPC)
- **Subnets**: Logical partitions of a VPC
  - **Public**: Has route to Internet Gateway
  - **Private**: No direct internet access, uses NAT Gateway for outbound
- **Route Tables**: Define where traffic goes

**Real-world example**:
```bash
# AWS CLI: Create VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# Create public subnet
aws ec2 create-subnet --vpc-id vpc-xxx --cidr-block 10.0.1.0/24

# Create internet gateway
aws ec2 create-internet-gateway
aws ec2 attach-internet-gateway --vpc-id vpc-xxx --internet-gateway-id igw-xxx
```

### 2. Connectivity Patterns

#### VPC Peering
Connect two VPCs directly (non-transitive).

```
VPC A (10.0.0.0/16) ←→ VPC B (10.1.0.0/16)
```

**Limitation**: If A peers with B, and B peers with C, A cannot talk to C.

#### Transit Gateway
Hub-and-spoke model for connecting many VPCs.

```
        Transit Gateway
       /      |      \
   VPC A   VPC B   VPC C
```

#### VPN / Direct Connect
Connect on-premise data centers to the cloud.

---

## 🐳 Container Networking

### Docker Networking

```bash
# List networks
docker network ls

# Default networks:
# - bridge: Default, isolated network
# - host: Use host's network directly
# - none: No networking

# Create custom network
docker network create myapp-network

# Run container on custom network
docker run --network myapp-network nginx

# Containers on same network can talk by name
docker run --network myapp-network --name web nginx
docker run --network myapp-network alpine ping web
```

**Network Drivers**:
- **bridge**: Default, isolated network
- **host**: Container uses host network (no isolation)
- **overlay**: Multi-host networking (Swarm/Kubernetes)
- **macvlan**: Assign MAC address to container
- **none**: No networking

---

## ☸️ Kubernetes Networking & CNI

### The Kubernetes Networking Model

**Three Rules**:
1. Every Pod gets its own IP address
2. Pods on any node can communicate with all pods on all other nodes without NAT
3. Agents on a node can communicate with all pods on that node

**How is this implemented?** **CNI (Container Network Interface)**

### CNI Plugins

| Plugin | How It Works | Use Case |
|:---|:---|:---|
| **Flannel** | Simple overlay network (VXLAN) | Easy setup, basic needs |
| **Calico** | Layer 3 routing (BGP) + Network Policies | Production, security |
| **Cilium** | eBPF-based high-performance | Advanced, observability |
| **Weave** | Mesh network | Multi-cloud |

**Real-world example**:
```bash
# Check CNI plugin
kubectl get pods -n kube-system | grep -E 'calico|flannel|cilium'

# View pod IP
kubectl get pod mypod -o wide
```

### Kubernetes Services

Services provide stable endpoints for pods.

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

**Service Types**:
- **ClusterIP**: Internal only (default)
- **NodePort**: Exposes on each node's IP at a static port
- **LoadBalancer**: Cloud provider load balancer
- **ExternalName**: DNS CNAME record

---

## 🕸️ Service Mesh (Istio/Linkerd)

When you have 100+ microservices, how do you manage traffic, security, and observability?

### The Sidecar Pattern

Instead of your app handling network logic (retries, mTLS, tracing), a lightweight proxy (sidecar) sits next to it.

```
┌─────────────────────────────┐
│          Pod                 │
│  ┌──────────┐  ┌──────────┐│
│  │   App    │  │  Envoy   ││
│  │  (8080)  │←→│ (Sidecar)││
│  └──────────┘  └────┬─────┘│
└──────────────────────┼──────┘
                       ↓
                  mTLS encrypted
                       ↓
                  Other pods
```

**Traffic Flow**:
1. **App A** sends request to **App B**
2. Request is intercepted by **Sidecar A** (Envoy proxy)
3. **Sidecar A** encrypts (mTLS) and routes to **Sidecar B**
4. **Sidecar B** decrypts and passes to **App B**

**Benefits**:
- **Observability**: Automatic tracing, metrics
- **Security**: mTLS everywhere (zero trust)
- **Traffic Control**: Canary deployments, circuit breaking, retries

**Real-world example (Istio)**:
```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
    - reviews
  http:
    - match:
        - headers:
            end-user:
              exact: jason
      route:
        - destination:
            host: reviews
            subset: v2
    - route:
        - destination:
            host: reviews
            subset: v1
```

---

## 🔐 DNS & TLS Deep Dive

### DNS Record Types

| Type | Purpose | Example |
|:---|:---|:---|
| **A** | IPv4 address | `example.com → 192.168.1.1` |
| **AAAA** | IPv6 address | `example.com → 2001:db8::1` |
| **CNAME** | Alias (cannot be at root) | `www.example.com → example.com` |
| **ALIAS/ANAME** | Root domain alias (cloud-specific) | `example.com → lb.aws.com` |
| **MX** | Mail server | `example.com → mail.example.com` |
| **TXT** | Text records (SPF, DKIM) | `example.com → "v=spf1..."` |

**Real-world debugging**:
```bash
# Trace DNS resolution
dig +trace example.com

# Query specific nameserver
dig @8.8.8.8 example.com

# Reverse DNS lookup
dig -x 8.8.8.8

# Check all records
dig example.com ANY
```

### TLS Handshake (Simplified)

```
Client                          Server
  │                               │
  │─── ClientHello ──────────────▶│
  │    (Supported ciphers)        │
  │                               │
  │◀── ServerHello ───────────────│
  │    (Chosen cipher)            │
  │◀── Certificate ───────────────│
  │    (Server's public key)      │
  │                               │
  │─── ClientKeyExchange ────────▶│
  │    (Encrypted session key)    │
  │                               │
  │◀── Finished ──────────────────│
  │                               │
  │─── Encrypted Data ───────────▶│
```

### Mutual TLS (mTLS)

The server **also** asks the client for a certificate. This is the foundation of **Zero Trust**.

**Use cases**:
- Service-to-service communication in Kubernetes
- API authentication
- Zero trust networks

---

## 🛠️ Network Troubleshooting

### 1. `curl` Like a Pro

Debug latency with `curl` timing.

```bash
# Create curl-format.txt
cat > curl-format.txt << 'EOF'
    time_namelookup:  %{time_namelookup}s\n
       time_connect:  %{time_connect}s\n
    time_appconnect:  %{time_appconnect}s\n
   time_pretransfer:  %{time_pretransfer}s\n
      time_redirect:  %{time_redirect}s\n
 time_starttransfer:  %{time_starttransfer}s\n
                    ----------\n
         time_total:  %{time_total}s\n
EOF

# Use it
curl -w "@curl-format.txt" -o /dev/null -s https://google.com
```

**Output**:
```
    time_namelookup:  0.005s   ← DNS lookup
       time_connect:  0.025s   ← TCP handshake
    time_appconnect:  0.085s   ← TLS handshake
 time_starttransfer:  0.120s   ← Time to first byte
         time_total:  0.150s   ← Total time
```

### 2. Debugging SSL/TLS

```bash
# Check certificate and chain
openssl s_client -connect google.com:443 -showcerts

# Check certificate expiration
echo | openssl s_client -connect google.com:443 2>/dev/null | openssl x509 -noout -dates

# Test specific TLS version
openssl s_client -connect google.com:443 -tls1_2
```

### 3. Packet Tracing

```bash
# Capture traffic on port 443
tcpdump -i eth0 port 443

# Save to file for Wireshark
tcpdump -i eth0 -w capture.pcap

# Filter by host
tcpdump -i eth0 host 192.168.1.100

# Trace route with MTR (better than traceroute)
mtr google.com
```

### 4. Common Issues

#### "Can't resolve hostname"
```bash
# Check DNS
dig example.com

# Check /etc/resolv.conf
cat /etc/resolv.conf

# Test with different DNS
dig @8.8.8.8 example.com
```

#### "Connection refused"
```bash
# Is service listening?
ss -tulpn | grep :80

# Is firewall blocking?
ufw status

# Can we connect locally?
curl http://localhost
```

#### "Slow network"
```bash
# Check latency
ping -c 10 google.com

# Check bandwidth
iperf3 -c server-ip

# Check packet loss
mtr google.com
```

---

## 🎯 CIDR Notation Quick Reference

| CIDR | Subnet Mask | Usable IPs | Use Case |
|:---|:---|:---|:---|
| /32 | 255.255.255.255 | 1 | Single host |
| /24 | 255.255.255.0 | 254 | Standard subnet |
| /20 | 255.255.240.0 | 4,094 | Large subnet |
| /16 | 255.255.0.0 | 65,534 | Standard VPC |
| /8 | 255.0.0.0 | 16,777,214 | Huge network |

**Calculate mentally**:
- `/24` = 256 IPs (2^8)
- `/20` = 4,096 IPs (2^12)
- `/16` = 65,536 IPs (2^16)

---

## ✅ Mastery Checklist

After mastering this guide, you should be able to:

- [ ] Calculate CIDR ranges mentally
- [ ] Configure a VPC with public/private subnets
- [ ] Understand VPC peering vs Transit Gateway
- [ ] Explain Docker networking modes
- [ ] Understand Kubernetes networking model and CNI
- [ ] Explain Service Mesh and sidecar pattern
- [ ] Debug DNS with `dig +trace`
- [ ] Understand TLS handshake and mTLS
- [ ] Troubleshoot network latency with `curl` timings
- [ ] Use `tcpdump` for packet analysis

---

## 📚 Practice Exercises

### Exercise 1: VPC Design
Design a 3-tier VPC:
- Web tier (public subnet)
- App tier (private subnet)
- Database tier (private subnet)
- NAT Gateway for outbound traffic

### Exercise 2: DNS Debugging
```bash
# Trace DNS resolution
dig +trace example.com

# Find authoritative nameservers
dig NS example.com

# Check DNSSEC
dig +dnssec example.com
```

### Exercise 3: Network Troubleshooting
Diagnose why a service is unreachable:
1. Check DNS resolution
2. Check if port is open
3. Check firewall rules
4. Check service logs
5. Test with curl timing

---

**Next Step**: Now that we have infrastructure (Linux) and connectivity (Networking), let's manage it with code. Master **[Git & GitOps](./git-gitops.md)**.

> **Remember**: Networking in the cloud is software-defined. You're not configuring routers; you're writing YAML and Terraform. Focus on understanding concepts, not memorizing commands.
