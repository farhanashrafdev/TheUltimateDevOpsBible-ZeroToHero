# System Design Interview: Complete Guide

## 🎯 Introduction

System design interviews test your ability to design scalable, reliable, and efficient systems. This guide covers the complete framework, methodologies, and real-world design examples you need to ace any system design interview.

### Why System Design Matters

- **Senior+ Roles**: Required for all senior engineering positions
- **Architecture Skills**: Tests your understanding of distributed systems
- **Trade-off Analysis**: Shows how you make engineering decisions
- **Communication**: Demonstrates how you collaborate with stakeholders

## 📚 The System Design Framework (RESHADED)

Use this framework for every system design interview:

```
R - Requirements (Functional & Non-Functional)
E - Estimation (Scale, Storage, Bandwidth)
S - Storage Schema (Database Design)
H - High-Level Design (Architecture)
A - API Design (Endpoints)
D - Detailed Design (Deep Dives)
E - Evaluation (Trade-offs)
D - Deployment & DevOps (Monitoring, Scaling)
```

### Step-by-Step Process

```
┌─────────────────────────────────────────────────────────────┐
│                    System Design Process                      │
│                                                               │
│  ┌──────────────┐  5 min                                     │
│  │ Requirements │ ← Clarify scope, users, features           │
│  └──────┬───────┘                                            │
│         │                                                     │
│         ▼                                                     │
│  ┌──────────────┐  5 min                                     │
│  │  Estimation  │ ← Calculate scale, storage, bandwidth      │
│  └──────┬───────┘                                            │
│         │                                                     │
│         ▼                                                     │
│  ┌──────────────┐  10 min                                    │
│  │  High-Level  │ ← Draw architecture, identify components   │
│  └──────┬───────┘                                            │
│         │                                                     │
│         ▼                                                     │
│  ┌──────────────┐  15 min                                    │
│  │   Detailed   │ ← Deep dive into critical components       │
│  └──────┬───────┘                                            │
│         │                                                     │
│         ▼                                                     │
│  ┌──────────────┐  5 min                                     │
│  │  Trade-offs  │ ← Discuss alternatives, limitations        │
│  └──────────────┘                                            │
└─────────────────────────────────────────────────────────────┘
```

## 📊 Back-of-Envelope Estimation

### Key Numbers to Memorize

```
Storage:
├── 1 char = 1 byte (ASCII) or 2-4 bytes (UTF-8)
├── 1 KB = 1,000 bytes (10^3)
├── 1 MB = 1,000 KB (10^6)
├── 1 GB = 1,000 MB (10^9)
├── 1 TB = 1,000 GB (10^12)
└── 1 PB = 1,000 TB (10^15)

Time:
├── 1 day = 86,400 seconds ≈ 100,000 seconds
├── 1 month = 2.5 million seconds
├── 1 year = 30 million seconds
└── 5 years = 150 million seconds

Traffic:
├── 1 million requests/day = ~12 requests/second
├── 1 billion requests/day = ~12,000 requests/second
└── Peak traffic = 2-3x average

Latency (2024 numbers):
├── L1 cache reference: 0.5 ns
├── L2 cache reference: 7 ns
├── Main memory reference: 100 ns
├── SSD random read: 150 μs
├── HDD seek: 10 ms
├── Network round trip (same datacenter): 0.5 ms
├── Network round trip (cross-continent): 150 ms
└── Read 1 MB from SSD: 1 ms
```

### Estimation Template

```
Given:
- DAU (Daily Active Users): X million
- Actions per user per day: Y
- Data size per action: Z KB

Calculate:
1. QPS (Queries Per Second)
   - Daily requests = DAU × actions = X × Y million
   - QPS = Daily requests / 86,400 ≈ Daily requests / 100,000
   - Peak QPS = QPS × 3

2. Storage (5 years)
   - Daily storage = Daily requests × data size
   - 5-year storage = Daily storage × 365 × 5

3. Bandwidth
   - Incoming = QPS × request size
   - Outgoing = QPS × response size
```

### Example: Twitter-like System

```
Given:
- 500M DAU
- 5 tweets viewed per user per day (reads)
- 0.1 tweets posted per user per day (writes)
- Average tweet size: 500 bytes (text + metadata)
- Average image: 500 KB

Read Traffic:
- Daily reads = 500M × 5 = 2.5B
- Read QPS = 2.5B / 100,000 = 25,000 QPS
- Peak Read QPS = 75,000 QPS

Write Traffic:
- Daily writes = 500M × 0.1 = 50M
- Write QPS = 50M / 100,000 = 500 QPS
- Peak Write QPS = 1,500 QPS

Storage (5 years):
- Text: 50M × 500B × 365 × 5 = 45 TB
- Images (20% with images): 10M × 500KB × 365 × 5 = 9 PB
- Total: ~10 PB

Bandwidth:
- Outgoing: 25,000 × 500B = 12.5 MB/s (text only)
- With images: Much higher, use CDN
```

## 🏗️ Core Building Blocks

### 1. Load Balancers

```
┌─────────────────────────────────────────────────────────┐
│                   Load Balancer Types                    │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  Layer 4 (Transport)          Layer 7 (Application)     │
│  ├── TCP/UDP level            ├── HTTP/HTTPS level      │
│  ├── Faster                   ├── More intelligent      │
│  ├── Less flexible            ├── Content-based routing │
│  └── Examples: AWS NLB        └── Examples: AWS ALB     │
│                                                          │
│  Algorithms:                                             │
│  ├── Round Robin: Equal distribution                    │
│  ├── Least Connections: Route to least busy            │
│  ├── IP Hash: Same client → same server                │
│  ├── Weighted: Based on server capacity                │
│  └── Random: Random selection                           │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### 2. Caching

```
┌─────────────────────────────────────────────────────────┐
│                    Caching Layers                        │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  Client ──► CDN ──► Load Balancer ──► App Server        │
│                                            │             │
│                                            ▼             │
│                                      Application Cache   │
│                                      (Redis/Memcached)   │
│                                            │             │
│                                            ▼             │
│                                        Database          │
│                                     (Query Cache)        │
│                                                          │
│  Cache Strategies:                                       │
│  ├── Cache-Aside: App manages cache                     │
│  ├── Read-Through: Cache manages reads                  │
│  ├── Write-Through: Write to cache + DB                 │
│  ├── Write-Behind: Write to cache, async to DB         │
│  └── Refresh-Ahead: Proactive refresh                   │
│                                                          │
│  Eviction Policies:                                      │
│  ├── LRU: Least Recently Used                          │
│  ├── LFU: Least Frequently Used                        │
│  ├── FIFO: First In First Out                          │
│  └── TTL: Time To Live                                  │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

**Cache-Aside Pattern (Most Common):**

```python
def get_user(user_id):
    # Try cache first
    user = cache.get(f"user:{user_id}")
    if user:
        return user  # Cache hit
    
    # Cache miss - get from DB
    user = db.query("SELECT * FROM users WHERE id = ?", user_id)
    
    # Store in cache for next time
    cache.set(f"user:{user_id}", user, ttl=3600)
    return user
```

### 3. Database Selection

```
┌─────────────────────────────────────────────────────────┐
│                   Database Selection                     │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  Relational (SQL)              NoSQL                     │
│  ├── ACID transactions         ├── Flexible schema       │
│  ├── Complex queries           ├── Horizontal scaling    │
│  ├── Strong consistency        ├── High throughput       │
│  └── Vertical scaling          └── Eventual consistency  │
│                                                          │
│  When to use SQL:              When to use NoSQL:        │
│  ├── Financial data            ├── User sessions        │
│  ├── User accounts             ├── Real-time analytics  │
│  ├── Inventory                 ├── IoT data             │
│  └── Complex relationships     └── Social feeds         │
│                                                          │
│  NoSQL Types:                                            │
│  ├── Document: MongoDB (flexible JSON documents)        │
│  ├── Key-Value: Redis, DynamoDB (simple lookups)       │
│  ├── Wide-Column: Cassandra (time-series, logs)        │
│  ├── Graph: Neo4j (relationships, social networks)     │
│  └── Search: Elasticsearch (full-text search)          │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### 4. Database Scaling

```
┌─────────────────────────────────────────────────────────┐
│                   Scaling Strategies                     │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  Replication:                                            │
│  ┌─────────┐     ┌─────────┐     ┌─────────┐           │
│  │ Primary │────▶│ Replica │────▶│ Replica │           │
│  │ (Write) │     │ (Read)  │     │ (Read)  │           │
│  └─────────┘     └─────────┘     └─────────┘           │
│                                                          │
│  Sharding (Horizontal Partitioning):                    │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐   │
│  │ Shard 1 │  │ Shard 2 │  │ Shard 3 │  │ Shard 4 │   │
│  │ A-F     │  │ G-L     │  │ M-R     │  │ S-Z     │   │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘   │
│                                                          │
│  Sharding Strategies:                                    │
│  ├── Range-based: user_id 1-1M, 1M-2M, etc.           │
│  ├── Hash-based: hash(user_id) % num_shards           │
│  ├── Directory-based: lookup table for shard mapping  │
│  └── Geographic: by region/country                     │
│                                                          │
│  Sharding Challenges:                                    │
│  ├── Cross-shard queries (joins)                       │
│  ├── Rebalancing when adding shards                    │
│  ├── Hotspots (uneven distribution)                    │
│  └── Referential integrity                             │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### 5. Message Queues

```
┌─────────────────────────────────────────────────────────┐
│                   Message Queue Patterns                 │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  Point-to-Point:                                         │
│  Producer ──► Queue ──► Consumer                        │
│                                                          │
│  Publish-Subscribe:                                      │
│  Producer ──► Topic ──┬──► Consumer 1                   │
│                       ├──► Consumer 2                   │
│                       └──► Consumer 3                   │
│                                                          │
│  Use Cases:                                              │
│  ├── Decoupling services                               │
│  ├── Async processing                                  │
│  ├── Load leveling (handle traffic spikes)            │
│  ├── Event sourcing                                    │
│  └── Reliable delivery                                 │
│                                                          │
│  Tools:                                                  │
│  ├── Kafka: High throughput, event streaming          │
│  ├── RabbitMQ: Flexible routing, AMQP                 │
│  ├── AWS SQS: Managed, serverless                     │
│  ├── Redis Pub/Sub: Fast, in-memory                   │
│  └── Apache Pulsar: Multi-tenancy                     │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### 6. CDN (Content Delivery Network)

```
┌─────────────────────────────────────────────────────────┐
│                         CDN                              │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  User (New York)                                         │
│       │                                                  │
│       ▼                                                  │
│  ┌─────────┐  Cache Miss   ┌─────────┐                 │
│  │ CDN Edge│──────────────▶│ Origin  │                 │
│  │ (NYC)   │◀──────────────│ Server  │                 │
│  └─────────┘  Fetch Content└─────────┘                 │
│       │                                                  │
│       ▼ Cache Hit (Fast!)                               │
│  User (New York)                                         │
│                                                          │
│  Benefits:                                               │
│  ├── Reduced latency (edge locations)                  │
│  ├── Reduced origin load                               │
│  ├── DDoS protection                                   │
│  └── HTTPS termination                                 │
│                                                          │
│  CDN Providers:                                          │
│  ├── CloudFlare                                         │
│  ├── AWS CloudFront                                    │
│  ├── Akamai                                            │
│  └── Fastly                                            │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

## 🔧 CAP Theorem & Consistency

### CAP Theorem

```
┌─────────────────────────────────────────────────────────┐
│                     CAP Theorem                          │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  You can only have 2 of 3:                              │
│                                                          │
│                    Consistency                           │
│                        /\                               │
│                       /  \                              │
│                      /    \                             │
│                     /  CA  \                            │
│                    /________\                           │
│                   /\        /\                          │
│                  /  \      /  \                         │
│                 / CP \    / AP \                        │
│                /______\  /______\                       │
│           Partition    Availability                     │
│           Tolerance                                      │
│                                                          │
│  CA: Traditional RDBMS (single node)                    │
│  CP: MongoDB, HBase (consistency over availability)     │
│  AP: Cassandra, DynamoDB (availability over consistency)│
│                                                          │
│  In distributed systems, P is mandatory, so choose:     │
│  ├── CP: When correctness is critical (banking)        │
│  └── AP: When availability is critical (social media)  │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### Consistency Levels

```
Strong Consistency:
- All nodes see the same data at the same time
- Higher latency
- Example: Bank transactions

Eventual Consistency:
- All nodes will eventually have the same data
- Lower latency, higher availability
- Example: Social media likes

Read-Your-Writes Consistency:
- User always sees their own writes
- Good balance for user-facing apps

Causal Consistency:
- Operations that are causally related are seen in order
- Example: Comment reply appears after original comment
```

## 📝 System Design Case Studies

---

### Design 1: URL Shortener (Like bit.ly)

#### Requirements

**Functional:**
- Shorten long URLs to short URLs
- Redirect short URL to original URL
- Custom short URLs (optional)
- Analytics (click count)
- URL expiration

**Non-Functional:**
- High availability (99.99%)
- Low latency (<100ms redirect)
- URLs should not be predictable

#### Estimation

```
Traffic:
- 100M URLs created per month
- Read:Write ratio = 100:1
- 10B redirects per month

QPS:
- Write: 100M / (30 × 86400) ≈ 40 URLs/second
- Read: 4000 redirects/second
- Peak: 12,000 redirects/second

Storage (5 years):
- 100M × 12 × 5 = 6B URLs
- Each URL: 500 bytes (long URL) + 7 bytes (short) + metadata
- Total: 6B × 600 bytes ≈ 3.6 TB

Bandwidth:
- Incoming: 40 × 500 bytes = 20 KB/s
- Outgoing: 4000 × 500 bytes = 2 MB/s
```

#### High-Level Design

```
┌─────────────────────────────────────────────────────────┐
│                   URL Shortener Architecture             │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  User ──► Load Balancer ──► API Servers                 │
│                                  │                       │
│           ┌──────────────────────┼──────────────────┐   │
│           │                      │                  │   │
│           ▼                      ▼                  ▼   │
│      ┌─────────┐          ┌─────────┐        ┌────────┐│
│      │  Cache  │          │Key Gen  │        │Analytics││
│      │ (Redis) │          │Service  │        │Service  ││
│      └─────────┘          └─────────┘        └────────┘│
│           │                      │                  │   │
│           └──────────────────────┼──────────────────┘   │
│                                  ▼                       │
│                           ┌──────────┐                  │
│                           │ Database │                  │
│                           │(Cassandra)│                  │
│                           └──────────┘                  │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

#### Database Schema

```sql
-- URLs Table
CREATE TABLE urls (
    short_url VARCHAR(7) PRIMARY KEY,
    long_url TEXT NOT NULL,
    user_id UUID,
    created_at TIMESTAMP DEFAULT NOW(),
    expires_at TIMESTAMP,
    click_count BIGINT DEFAULT 0
);

-- Index for user lookups
CREATE INDEX idx_user_id ON urls(user_id);
```

#### Key Generation

```python
# Option 1: Base62 encoding (a-z, A-Z, 0-9)
# 62^7 = 3.5 trillion combinations

import hashlib
import base64

def generate_short_url(long_url):
    # MD5 hash of URL
    hash_obj = hashlib.md5(long_url.encode())
    hash_bytes = hash_obj.digest()
    
    # Base64 encode and take first 7 characters
    encoded = base64.urlsafe_b64encode(hash_bytes).decode()
    return encoded[:7]

# Option 2: Pre-generated Key Service (Better)
class KeyGenerationService:
    def __init__(self):
        self.unused_keys = []  # Pre-generated keys
        self.used_keys = set()
    
    def get_key(self):
        if not self.unused_keys:
            self._generate_keys()
        key = self.unused_keys.pop()
        self.used_keys.add(key)
        return key
    
    def _generate_keys(self):
        # Generate batch of unique keys
        # Store in database for persistence
        pass
```

#### API Design

```yaml
# Create short URL
POST /api/v1/urls
Request:
  {
    "long_url": "https://example.com/very/long/url",
    "custom_alias": "mylink",  # optional
    "expires_at": "2025-12-31"  # optional
  }
Response:
  {
    "short_url": "https://short.ly/abc1234",
    "long_url": "https://example.com/very/long/url",
    "expires_at": "2025-12-31"
  }

# Redirect (GET)
GET /{short_url}
Response: 301 Redirect to long_url

# Get analytics
GET /api/v1/urls/{short_url}/stats
Response:
  {
    "short_url": "abc1234",
    "click_count": 1234,
    "created_at": "2024-01-15",
    "referrers": {...}
  }
```

#### Detailed Design

**Read Flow (Redirect):**
```
1. User requests short.ly/abc1234
2. Load balancer routes to API server
3. API server checks Redis cache
4. If cache hit → return long URL
5. If cache miss → query Cassandra
6. Store in cache, return long URL
7. Async: increment click count
```

**Write Flow:**
```
1. User submits long URL
2. Check if URL already exists (optional dedup)
3. Get unique key from Key Generation Service
4. Store in Cassandra
5. Invalidate any related cache
6. Return short URL
```

---

### Design 2: Twitter/X Timeline

#### Requirements

**Functional:**
- Post tweets (text, images, videos)
- Follow/unfollow users
- Home timeline (tweets from followed users)
- User timeline (user's own tweets)
- Like, retweet, reply

**Non-Functional:**
- 500M DAU
- Timeline load < 200ms
- Eventual consistency acceptable
- High availability

#### Estimation

```
Users:
- 500M DAU
- Average follows: 200 users
- Average followers: 200 users
- Celebrity accounts: 50M+ followers

Traffic:
- Timeline refreshes: 500M × 10/day = 5B/day
- Timeline QPS: 5B / 86400 ≈ 60,000 QPS
- Tweet writes: 500M × 0.5/day = 250M/day
- Write QPS: 3,000 QPS

Storage:
- Tweet: 280 chars + metadata ≈ 500 bytes
- Daily tweets: 250M × 500B = 125 GB
- 5 years: 225 TB (text only)
- With media: 10+ PB
```

#### High-Level Design

```
┌─────────────────────────────────────────────────────────┐
│                  Twitter Architecture                    │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │                    Client Apps                    │   │
│  └──────────────────────────────────────────────────┘   │
│                          │                               │
│                          ▼                               │
│  ┌──────────────────────────────────────────────────┐   │
│  │              Load Balancer / API Gateway          │   │
│  └──────────────────────────────────────────────────┘   │
│                          │                               │
│         ┌────────────────┼────────────────┐             │
│         ▼                ▼                ▼             │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐        │
│  │   Tweet    │  │  Timeline  │  │   User     │        │
│  │  Service   │  │  Service   │  │  Service   │        │
│  └────────────┘  └────────────┘  └────────────┘        │
│         │                │                │             │
│         ▼                ▼                ▼             │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐        │
│  │Tweet Store │  │Timeline    │  │User Store  │        │
│  │(Cassandra) │  │Cache(Redis)│  │  (MySQL)   │        │
│  └────────────┘  └────────────┘  └────────────┘        │
│         │                                               │
│         ▼                                               │
│  ┌──────────────────────────────────────────────────┐   │
│  │              Fanout Service (Kafka)               │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

#### Timeline Generation Approaches

**1. Pull Model (Fan-out on Read)**
```
When user loads timeline:
1. Get list of followed users
2. Fetch recent tweets from each user
3. Merge and sort by time
4. Return top N tweets

Pros: Simple, less storage
Cons: Slow for users following many accounts
```

**2. Push Model (Fan-out on Write)**
```
When user posts tweet:
1. Get list of followers
2. Push tweet ID to each follower's timeline cache
3. When user loads timeline, just read from cache

Pros: Fast reads
Cons: Celebrity problem (50M followers = 50M writes)
```

**3. Hybrid Approach (Twitter's actual approach)**
```
- Regular users: Push model (fanout on write)
- Celebrities (>10K followers): Pull model (fanout on read)

When loading timeline:
1. Fetch pre-computed timeline from cache (pushed tweets)
2. Fetch tweets from celebrities user follows
3. Merge and return
```

#### Data Models

```sql
-- Users Table (MySQL - strong consistency for user data)
CREATE TABLE users (
    user_id BIGINT PRIMARY KEY,
    username VARCHAR(50) UNIQUE,
    email VARCHAR(255),
    created_at TIMESTAMP,
    follower_count INT,
    following_count INT
);

-- Follows Table (MySQL)
CREATE TABLE follows (
    follower_id BIGINT,
    followee_id BIGINT,
    created_at TIMESTAMP,
    PRIMARY KEY (follower_id, followee_id)
);
CREATE INDEX idx_followee ON follows(followee_id);

-- Tweets Table (Cassandra - high write throughput)
CREATE TABLE tweets (
    tweet_id BIGINT,
    user_id BIGINT,
    content TEXT,
    media_urls LIST<TEXT>,
    created_at TIMESTAMP,
    like_count INT,
    retweet_count INT,
    PRIMARY KEY (tweet_id)
);

-- User Timeline (Cassandra)
CREATE TABLE user_timeline (
    user_id BIGINT,
    tweet_id BIGINT,
    created_at TIMESTAMP,
    PRIMARY KEY (user_id, created_at)
) WITH CLUSTERING ORDER BY (created_at DESC);

-- Home Timeline Cache (Redis)
-- Key: home_timeline:{user_id}
-- Value: List of tweet_ids (sorted by time)
```

#### Tweet Posting Flow

```
1. User posts tweet
2. Tweet Service:
   - Validate content
   - Store in Tweets table
   - Upload media to object storage (S3)
   - Publish to Kafka topic "new_tweets"

3. Fanout Service (Kafka consumer):
   - Check if user is celebrity (>10K followers)
   - If not celebrity:
     - Get follower list
     - For each follower, push tweet_id to their timeline cache
   - If celebrity:
     - Don't fanout (will be pulled on read)

4. Analytics Service:
   - Update trending topics
   - Track engagement metrics
```

---

### Design 3: Distributed Rate Limiter

#### Requirements

**Functional:**
- Limit requests per user/IP/API key
- Multiple rate limit rules (per second, minute, hour)
- Return meaningful error when limited

**Non-Functional:**
- Very low latency (<5ms)
- Distributed (works across multiple servers)
- Accurate (no significant over-limiting)

#### Algorithms

**1. Token Bucket**
```
┌─────────────────────────────────────────────────────────┐
│                    Token Bucket                          │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  Bucket Capacity: 10 tokens                             │
│  Refill Rate: 2 tokens/second                           │
│                                                          │
│  ┌─────────────────────────┐                            │
│  │  🪙 🪙 🪙 🪙 🪙 🪙        │ ← Current: 6 tokens       │
│  │                         │                            │
│  │      Token Bucket       │                            │
│  └─────────────────────────┘                            │
│           │                                              │
│           ▼                                              │
│  Request arrives → Take 1 token                         │
│  If tokens available → Allow request                    │
│  If no tokens → Reject (429 Too Many Requests)         │
│                                                          │
│  Pros: Allows burst traffic, smooth rate limiting       │
│  Cons: Need to track last refill time                  │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

```python
class TokenBucket:
    def __init__(self, capacity, refill_rate):
        self.capacity = capacity
        self.tokens = capacity
        self.refill_rate = refill_rate  # tokens per second
        self.last_refill = time.time()
    
    def allow_request(self):
        self._refill()
        if self.tokens >= 1:
            self.tokens -= 1
            return True
        return False
    
    def _refill(self):
        now = time.time()
        elapsed = now - self.last_refill
        tokens_to_add = elapsed * self.refill_rate
        self.tokens = min(self.capacity, self.tokens + tokens_to_add)
        self.last_refill = now
```

**2. Sliding Window Log**
```python
class SlidingWindowLog:
    def __init__(self, window_size, max_requests):
        self.window_size = window_size  # seconds
        self.max_requests = max_requests
        self.request_log = []
    
    def allow_request(self):
        now = time.time()
        window_start = now - self.window_size
        
        # Remove old requests outside window
        self.request_log = [t for t in self.request_log if t > window_start]
        
        if len(self.request_log) < self.max_requests:
            self.request_log.append(now)
            return True
        return False
```

**3. Sliding Window Counter (Best for distributed)**
```python
class SlidingWindowCounter:
    def __init__(self, redis_client, window_size, max_requests):
        self.redis = redis_client
        self.window_size = window_size
        self.max_requests = max_requests
    
    def allow_request(self, user_id):
        now = time.time()
        current_window = int(now // self.window_size)
        previous_window = current_window - 1
        
        # Keys for current and previous windows
        current_key = f"rate:{user_id}:{current_window}"
        previous_key = f"rate:{user_id}:{previous_window}"
        
        # Get counts
        current_count = int(self.redis.get(current_key) or 0)
        previous_count = int(self.redis.get(previous_key) or 0)
        
        # Calculate weighted count
        elapsed_in_window = now % self.window_size
        weight = elapsed_in_window / self.window_size
        weighted_count = current_count + previous_count * (1 - weight)
        
        if weighted_count < self.max_requests:
            # Increment current window
            pipe = self.redis.pipeline()
            pipe.incr(current_key)
            pipe.expire(current_key, self.window_size * 2)
            pipe.execute()
            return True
        return False
```

#### Distributed Architecture

```
┌─────────────────────────────────────────────────────────┐
│              Distributed Rate Limiter                    │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  Client                                                  │
│     │                                                    │
│     ▼                                                    │
│  ┌─────────────────────────────────────────────────┐    │
│  │              API Gateway                         │    │
│  │         (Rate Limit Check)                      │    │
│  └─────────────────────────────────────────────────┘    │
│     │                                                    │
│     ▼                                                    │
│  ┌─────────────────────────────────────────────────┐    │
│  │           Redis Cluster                          │    │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐         │    │
│  │  │ Node 1  │  │ Node 2  │  │ Node 3  │         │    │
│  │  │user:1-99│  │user:100-│  │user:200+│         │    │
│  │  │  counts │  │199 counts│  │  counts │         │    │
│  │  └─────────┘  └─────────┘  └─────────┘         │    │
│  └─────────────────────────────────────────────────┘    │
│     │                                                    │
│     ▼ (If allowed)                                      │
│  ┌─────────────────────────────────────────────────┐    │
│  │              Backend Services                    │    │
│  └─────────────────────────────────────────────────┘    │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

### Design 4: Chat System (WhatsApp/Slack)

#### Requirements

**Functional:**
- 1:1 messaging
- Group chat (up to 500 members)
- Online/offline status
- Read receipts
- Media sharing
- Message history

**Non-Functional:**
- Real-time delivery (<100ms)
- Message ordering guaranteed
- High availability
- End-to-end encryption (optional)

#### Estimation

```
Scale:
- 1B users, 100M DAU
- Average messages: 50/user/day
- Daily messages: 5B
- QPS: 60,000 messages/second

Storage:
- Message size: 1KB average
- Daily: 5B × 1KB = 5TB
- 5 years: 9 PB
```

#### High-Level Design

```
┌─────────────────────────────────────────────────────────┐
│                  Chat System Architecture                │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌──────────┐                           ┌──────────┐    │
│  │  User A  │                           │  User B  │    │
│  │ (Mobile) │                           │  (Web)   │    │
│  └────┬─────┘                           └────┬─────┘    │
│       │                                      │          │
│       │ WebSocket                   WebSocket│          │
│       │                                      │          │
│       ▼                                      ▼          │
│  ┌──────────────────────────────────────────────────┐   │
│  │               WebSocket Servers                   │   │
│  │         (Stateful - maintains connections)        │   │
│  └──────────────────────────────────────────────────┘   │
│                          │                               │
│         ┌────────────────┼────────────────┐             │
│         ▼                ▼                ▼             │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐        │
│  │   Chat     │  │  Presence  │  │   Group    │        │
│  │  Service   │  │  Service   │  │  Service   │        │
│  └────────────┘  └────────────┘  └────────────┘        │
│         │                │                │             │
│         ▼                ▼                ▼             │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐        │
│  │  Message   │  │   Redis    │  │   Group    │        │
│  │   Store    │  │  (Online   │  │   Store    │        │
│  │(Cassandra) │  │   Status)  │  │  (MySQL)   │        │
│  └────────────┘  └────────────┘  └────────────┘        │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │          Message Queue (Kafka)                    │   │
│  │    (For async delivery, offline messages)         │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

#### Message Flow

```
1:1 Message Flow:
                                                          
  User A                Chat Server              User B    
    │                       │                       │      
    │──── Send Message ────▶│                       │      
    │                       │                       │      
    │                       │── Store in DB ───────▶│      
    │                       │                       │      
    │                       │── Is User B online? ─▶│      
    │                       │                       │      
    │                       │   If online:          │      
    │                       │──── Push via WS ─────▶│      
    │                       │                       │      
    │                       │   If offline:         │      
    │                       │── Queue + Push Notif ─│      
    │                       │                       │      
    │◀── Delivery ACK ──────│                       │      
    │                       │                       │      
    │                       │◀──── Read Receipt ────│      
    │◀── Read Receipt ──────│                       │      
    │                       │                       │      
```

#### Data Models

```sql
-- Messages (Cassandra - optimized for writes)
CREATE TABLE messages (
    conversation_id UUID,
    message_id TIMEUUID,
    sender_id BIGINT,
    content TEXT,
    media_url TEXT,
    message_type VARCHAR(20),  -- text, image, video
    created_at TIMESTAMP,
    PRIMARY KEY (conversation_id, message_id)
) WITH CLUSTERING ORDER BY (message_id DESC);

-- Conversations (MySQL)
CREATE TABLE conversations (
    conversation_id UUID PRIMARY KEY,
    type ENUM('direct', 'group'),
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);

-- Conversation Participants
CREATE TABLE conversation_participants (
    conversation_id UUID,
    user_id BIGINT,
    joined_at TIMESTAMP,
    last_read_message_id TIMEUUID,
    PRIMARY KEY (conversation_id, user_id)
);

-- Groups (MySQL)
CREATE TABLE groups (
    group_id UUID PRIMARY KEY,
    name VARCHAR(255),
    creator_id BIGINT,
    created_at TIMESTAMP,
    member_count INT
);
```

#### WebSocket Connection Management

```python
class WebSocketManager:
    def __init__(self):
        # Map: user_id -> WebSocket connection
        self.connections = {}
        # Map: user_id -> server_id (for distributed)
        self.user_server_map = {}  # Stored in Redis
    
    async def connect(self, user_id, websocket):
        self.connections[user_id] = websocket
        # Register in Redis: user_id -> this server
        await redis.set(f"user:server:{user_id}", SERVER_ID)
        await self.broadcast_online_status(user_id, True)
    
    async def disconnect(self, user_id):
        del self.connections[user_id]
        await redis.delete(f"user:server:{user_id}")
        await self.broadcast_online_status(user_id, False)
    
    async def send_message(self, user_id, message):
        if user_id in self.connections:
            # User connected to this server
            await self.connections[user_id].send(message)
        else:
            # User on different server - route via message queue
            server_id = await redis.get(f"user:server:{user_id}")
            if server_id:
                await kafka.send(f"messages.{server_id}", message)
            else:
                # User offline - store for later
                await self.store_offline_message(user_id, message)
```

---

### Design 5: Notification System

#### Requirements

**Functional:**
- Push notifications (mobile)
- SMS notifications
- Email notifications
- In-app notifications
- User preferences (opt-in/out)
- Rate limiting per user

**Non-Functional:**
- 100M notifications/day
- Delivery within 5 seconds
- At-least-once delivery
- Prioritization (urgent vs normal)

#### High-Level Design

```
┌─────────────────────────────────────────────────────────┐
│              Notification System Architecture            │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  Services (Tweet, Order, etc.)                          │
│         │                                                │
│         ▼ Notification Request                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │            Notification Service                   │   │
│  │  ┌──────────────────────────────────────────┐    │   │
│  │  │         Validation & Enrichment          │    │   │
│  │  │  • Check user preferences                │    │   │
│  │  │  • Rate limiting                         │    │   │
│  │  │  • Template rendering                    │    │   │
│  │  └──────────────────────────────────────────┘    │   │
│  └──────────────────────────────────────────────────┘   │
│                          │                               │
│                          ▼                               │
│  ┌──────────────────────────────────────────────────┐   │
│  │         Message Queue (Kafka)                     │   │
│  │  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐    │   │
│  │  │ High   │ │ Normal │ │  Low   │ │  Bulk  │    │   │
│  │  │Priority│ │Priority│ │Priority│ │        │    │   │
│  │  └────────┘ └────────┘ └────────┘ └────────┘    │   │
│  └──────────────────────────────────────────────────┘   │
│                          │                               │
│         ┌────────────────┼────────────────┐             │
│         ▼                ▼                ▼             │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐        │
│  │   Push     │  │   SMS      │  │   Email    │        │
│  │  Worker    │  │  Worker    │  │  Worker    │        │
│  └────────────┘  └────────────┘  └────────────┘        │
│         │                │                │             │
│         ▼                ▼                ▼             │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐        │
│  │   APNs     │  │  Twilio    │  │ SendGrid   │        │
│  │   FCM      │  │            │  │            │        │
│  └────────────┘  └────────────┘  └────────────┘        │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## 🎯 Common Interview Topics Quick Reference

### Scalability Patterns

```
┌─────────────────────────────────────────────────────────┐
│                 Scalability Patterns                     │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  Horizontal Scaling:                                     │
│  ├── Add more servers                                   │
│  ├── Stateless services                                 │
│  ├── Load balancing                                     │
│  └── Database sharding                                  │
│                                                          │
│  Vertical Scaling:                                       │
│  ├── Bigger servers (more CPU, RAM)                    │
│  ├── SSD storage                                        │
│  └── Easier but has limits                             │
│                                                          │
│  Caching:                                                │
│  ├── Application cache (Redis)                         │
│  ├── Database query cache                              │
│  ├── CDN for static content                            │
│  └── Browser caching                                   │
│                                                          │
│  Async Processing:                                       │
│  ├── Message queues                                    │
│  ├── Background jobs                                   │
│  └── Event-driven architecture                         │
│                                                          │
│  Database Optimization:                                  │
│  ├── Indexing                                          │
│  ├── Query optimization                                │
│  ├── Read replicas                                     │
│  ├── Sharding                                          │
│  └── Connection pooling                                │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### Reliability Patterns

```
┌─────────────────────────────────────────────────────────┐
│                  Reliability Patterns                    │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  Redundancy:                                             │
│  ├── Multiple instances                                │
│  ├── Multi-region deployment                           │
│  ├── Database replicas                                 │
│  └── Backup systems                                    │
│                                                          │
│  Fault Tolerance:                                        │
│  ├── Circuit breakers                                  │
│  ├── Retry with exponential backoff                   │
│  ├── Timeouts                                          │
│  ├── Bulkheads (isolation)                            │
│  └── Graceful degradation                             │
│                                                          │
│  Monitoring & Alerting:                                  │
│  ├── Health checks                                     │
│  ├── Metrics collection                                │
│  ├── Log aggregation                                   │
│  ├── Distributed tracing                               │
│  └── Anomaly detection                                 │
│                                                          │
│  Recovery:                                               │
│  ├── Automated failover                                │
│  ├── Rollback capability                               │
│  ├── Backup and restore                                │
│  └── Disaster recovery plan                            │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### Security Considerations

```
Always discuss security in interviews:

1. Authentication & Authorization
   - OAuth 2.0 / JWT
   - API keys
   - Role-based access control (RBAC)

2. Data Protection
   - Encryption at rest (AES-256)
   - Encryption in transit (TLS 1.3)
   - Key management (KMS)

3. Network Security
   - VPC / private networks
   - Firewall rules
   - DDoS protection

4. Input Validation
   - SQL injection prevention
   - XSS prevention
   - Rate limiting

5. Audit & Compliance
   - Audit logging
   - GDPR / HIPAA compliance
   - Data retention policies
```

## ✅ Interview Checklist

Before your interview:
- [ ] Practice estimation calculations
- [ ] Know the RESHADED framework
- [ ] Understand CAP theorem
- [ ] Know when to use SQL vs NoSQL
- [ ] Understand caching strategies
- [ ] Know message queue use cases
- [ ] Practice drawing architecture diagrams
- [ ] Prepare to discuss trade-offs

During the interview:
- [ ] Clarify requirements first (5 min)
- [ ] Do back-of-envelope estimation
- [ ] Start with high-level design
- [ ] Dive deep into 2-3 components
- [ ] Discuss trade-offs and alternatives
- [ ] Address scalability and reliability
- [ ] Mention monitoring and security

## 📚 Additional System Design Topics

### Design Problems to Practice

1. **Storage Systems**: Design Dropbox, Google Drive
2. **Search Systems**: Design Google Search, Elasticsearch
3. **Streaming**: Design Netflix, YouTube
4. **E-commerce**: Design Amazon, payment system
5. **Social**: Design Facebook, Instagram, LinkedIn
6. **Messaging**: Design WhatsApp, Slack, Discord
7. **Location**: Design Uber, Google Maps
8. **Monitoring**: Design Datadog, metrics system
9. **Ad System**: Design ad serving, click tracking
10. **Ticketing**: Design Ticketmaster, booking system

### Resources for Further Study

- "Designing Data-Intensive Applications" by Martin Kleppmann
- System Design Primer (GitHub)
- Grokking the System Design Interview
- Company engineering blogs (Netflix, Uber, Twitter)

---

**Remember**: System design interviews are about demonstrating your thought process. There's rarely a "perfect" answer - interviewers want to see how you approach problems, make trade-offs, and communicate your ideas clearly.

**Pro Tips**:
1. Always start with requirements - don't jump into design
2. Use back-of-envelope math to justify decisions
3. Think out loud - let the interviewer follow your reasoning
4. Be ready to pivot if the interviewer guides you differently
5. Discuss monitoring, alerting, and operational concerns
6. Security should be mentioned for every system

