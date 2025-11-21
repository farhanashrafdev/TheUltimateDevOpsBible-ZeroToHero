# System Design Interview: Complete Guide

## 🎯 Introduction

System design interviews test your ability to design scalable, reliable systems. This guide covers the design process and common questions.

## 📋 Design Process

### 1. Requirements Clarification

**Ask Questions**:
- Functional requirements (what the system does)
- Non-functional requirements (scale, performance, availability)
- Constraints (budget, time, resources)
- Assumptions

**Example Questions**:
- How many users?
- Read/write ratio?
- Data size?
- Latency requirements?
- Availability requirements?

### 2. High-Level Design

**Components**:
- Identify major components
- Define APIs
- Data flow
- System architecture

**Diagram**:
```
Client → Load Balancer → Web Servers → Application Servers → Database
                              ↓
                         Cache Layer
                              ↓
                         Message Queue
```

### 3. Detailed Design

**Deep Dive**:
- Database schema
- Algorithms
- Data partitioning
- Caching strategy
- Scaling strategy

### 4. Trade-offs

**Discuss**:
- Performance vs. cost
- Consistency vs. availability
- Latency vs. throughput
- Complexity vs. simplicity

## 🏗️ Common Design Questions

### Design a URL Shortener (like bit.ly)

**Requirements**:
- Shorten long URLs
- Redirect to original URL
- Analytics
- Scale: 100M URLs/day

**High-Level Design**:
```
Client → Load Balancer → Web Servers → Application Servers
                                              ↓
                                         Database
                                              ↓
                                         Cache (Redis)
```

**Key Components**:
1. **URL Encoding**: Base62 encoding (a-z, A-Z, 0-9)
2. **Database**: Store short URL → long URL mapping
3. **Cache**: Redis for hot URLs
4. **Load Balancer**: Distribute traffic

**Detailed Design**:
- **Database Schema**:
  - short_url (PK)
  - long_url
  - created_at
  - expires_at
  - click_count

- **Algorithm**:
  - Hash long URL (MD5/SHA256)
  - Encode to Base62
  - Check for collisions
  - Store in database

- **Scaling**:
  - Database sharding (by short URL)
  - Cache frequently accessed URLs
  - CDN for static assets

**Trade-offs**:
- Database vs. Cache: Use both
- Consistency: Eventual consistency acceptable
- Availability: High availability required

### Design a Chat System (like WhatsApp)

**Requirements**:
- Send/receive messages
- Real-time delivery
- Group chats
- Online status
- Scale: 1B users, 50M concurrent

**High-Level Design**:
```
Client → Load Balancer → Chat Servers → Message Queue
                                            ↓
                                       Database
                                            ↓
                                       Cache
```

**Key Components**:
1. **Chat Servers**: Handle connections (WebSocket)
2. **Message Queue**: Kafka for message delivery
3. **Database**: Store messages
4. **Cache**: Online status, recent messages

**Detailed Design**:
- **Message Flow**:
  1. User sends message
  2. Chat server receives
  3. Store in database
  4. Publish to message queue
  5. Deliver to recipient(s)

- **Online Status**:
  - Redis for online users
  - Heartbeat mechanism
  - TTL for offline detection

- **Scaling**:
  - Horizontal scaling of chat servers
  - Database sharding by user_id
  - Message queue partitioning

**Trade-offs**:
- Real-time vs. Reliability: At-least-once delivery
- Consistency: Eventual consistency
- Latency: Optimize for low latency

### Design a Monitoring System (like Datadog)

**Requirements**:
- Collect metrics from millions of servers
- Store time-series data
- Query and visualize
- Alerting
- Scale: 1M metrics/second

**High-Level Design**:
```
Agents → Collectors → Message Queue → Storage → Query Engine
                                            ↓
                                       Dashboards
                                            ↓
                                       Alerting
```

**Key Components**:
1. **Agents**: Collect metrics from servers
2. **Collectors**: Aggregate and forward
3. **Message Queue**: Kafka for buffering
4. **Storage**: Time-series database (InfluxDB, TimescaleDB)
5. **Query Engine**: Process queries
6. **Dashboards**: Visualization

**Detailed Design**:
- **Data Model**:
  - Metric name
  - Tags (key-value pairs)
  - Timestamp
  - Value

- **Storage**:
  - Time-series database
  - Compression
  - Retention policies
  - Downsampling

- **Scaling**:
  - Horizontal scaling
  - Data partitioning by time
  - Caching frequent queries

**Trade-offs**:
- Storage vs. Query Speed: Use compression
- Accuracy vs. Storage: Downsampling
- Latency vs. Throughput: Batch processing

### Design a Distributed Cache (like Redis Cluster)

**Requirements**:
- Key-value store
- High availability
- Low latency
- Scale: 1B keys, 1M ops/sec

**High-Level Design**:
```
Clients → Load Balancer → Cache Nodes → Replication
```

**Key Components**:
1. **Cache Nodes**: Store data
2. **Consistent Hashing**: Key distribution
3. **Replication**: High availability
4. **Eviction Policy**: LRU, LFU

**Detailed Design**:
- **Sharding**:
  - Consistent hashing
  - Virtual nodes
  - Rebalancing

- **Replication**:
  - Master-slave replication
  - Automatic failover
  - Read from replicas

- **Eviction**:
  - LRU (Least Recently Used)
  - TTL expiration
  - Memory limits

**Trade-offs**:
- Consistency vs. Availability: Eventual consistency
- Memory vs. Performance: Use SSD for overflow
- Latency vs. Durability: Async replication

## 🎯 Design Principles

### Scalability

**Horizontal Scaling**:
- Add more servers
- Stateless services
- Load balancing

**Vertical Scaling**:
- Increase server capacity
- Limited by hardware

**Best Practice**: Design for horizontal scaling

### Reliability

**Redundancy**:
- Multiple instances
- Multi-region
- Replication

**Fault Tolerance**:
- Circuit breakers
- Retries
- Graceful degradation

### Performance

**Caching**:
- Application cache
- CDN
- Database cache

**Database Optimization**:
- Indexing
- Query optimization
- Read replicas

**CDN**:
- Static content
- Global distribution
- Reduced latency

### Security

**Authentication**:
- OAuth, JWT
- API keys
- Multi-factor auth

**Authorization**:
- RBAC
- Least privilege
- Access control

**Data Protection**:
- Encryption (at rest, in transit)
- Secrets management
- Audit logging

## 📊 Common Patterns

### Microservices

**Benefits**:
- Independent scaling
- Technology diversity
- Fault isolation

**Challenges**:
- Service communication
- Data consistency
- Distributed tracing

### Event-Driven Architecture

**Components**:
- Event producers
- Message queue
- Event consumers

**Benefits**:
- Loose coupling
- Scalability
- Resilience

### CQRS (Command Query Responsibility Segregation)

**Concept**:
- Separate read and write models
- Optimize each independently

**Use Cases**:
- High read/write ratio
- Complex queries
- Event sourcing

## ✅ Interview Tips

1. **Clarify Requirements**: Ask questions
2. **Think Out Loud**: Explain your thinking
3. **Start Simple**: Basic design first
4. **Iterate**: Add details gradually
5. **Discuss Trade-offs**: Show understanding
6. **Consider Scale**: Think about millions of users
7. **Security**: Don't forget security
8. **Monitoring**: Include observability

## 🎯 Key Areas to Master

- [ ] Scalability patterns
- [ ] Database design
- [ ] Caching strategies
- [ ] Load balancing
- [ ] Message queues
- [ ] Distributed systems
- [ ] Trade-offs
- [ ] System architecture
- [ ] Performance optimization
- [ ] Security considerations

---

**Remember**: System design interviews test your ability to think through complex problems. Start with requirements, design high-level, then dive deep. Always discuss trade-offs and consider scale, reliability, and security.
