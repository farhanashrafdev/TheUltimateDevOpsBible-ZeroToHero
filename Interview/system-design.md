# System Design Interview Questions (DevOps Focus)

## 🎯 Introduction
System Design interviews test your ability to design scalable, reliable, and maintainable systems.

## 📐 Framework: The "PEDALS" Method
1. **P**rocess Requirements (Functional/Non-functional)
2. **E**stimation (Traffic, Storage)
3. **D**esign High-Level (Diagram)
4. **A**PI Design
5. **L**ow-Level Details (Schema, Algorithms)
6. **S**cale (Caching, Sharding, Load Balancing)

---

## ❓ Common Questions

### 1. Design a URL Shortener (TinyURL)
**Key DevOps Aspects**:
- **Read vs Write**: Read heavy (100:1).
- **Database**: NoSQL (DynamoDB/Cassandra) for speed and scale. Key=ShortURL, Value=LongURL.
- **Caching**: Redis for hot URLs.
- **Hashing**: Base62 encoding of unique ID.
- **High Availability**: Multi-region deployment.

### 2. Design a Log Aggregation System
**Key DevOps Aspects**:
- **Ingestion**: High write throughput. Kafka as buffer.
- **Agent**: Fluentd/Filebeat on nodes.
- **Storage**: Elasticsearch (Hot/Warm/Cold architecture).
- **Retention**: Archive old logs to S3 (Glacier).
- **Query**: Kibana.

### 3. Design a CI/CD Platform (like GitHub Actions)
**Key DevOps Aspects**:
- **Isolation**: Build agents must be isolated (Firecracker VMs / Containers).
- **Scheduling**: Queue system (Redis/SQS) to distribute jobs.
- **Storage**: Artifact storage (S3) with lifecycle policies.
- **Security**: Secrets management (Vault).

### 4. Design Instagram (Photo Sharing)
**Key DevOps Aspects**:
- **Storage**: Object storage (S3) for photos + CDN (CloudFront).
- **Database**: Relational (PostgreSQL) for users/metadata. Sharding by UserID.
- **Feed Generation**: Pre-compute feeds (Fan-out on write) vs Pull on read.

---

## 🏗️ Key Concepts to Master

### 1. Load Balancing
- **L4 vs L7**: TCP vs HTTP.
- **Algorithms**: Round Robin, Least Connections, IP Hash.

### 2. Caching
- **Strategies**: Cache-Aside, Write-Through, Write-Back.
- **Eviction**: LRU, LFU.
- **CDN**: Edge caching for static assets.

### 3. Database Scaling
- **Vertical**: Bigger instance.
- **Horizontal**: Read Replicas.
- **Sharding**: Partitioning data across nodes.

### 4. Asynchronous Processing
- **Message Queues**: Kafka, RabbitMQ, SQS.
- **Decoupling**: Producer -> Queue -> Consumer.

### 5. High Availability
- **Active-Active**: Traffic to both regions.
- **Active-Passive**: Failover to standby.
- **DR**: RPO (Data loss) vs RTO (Downtime).
