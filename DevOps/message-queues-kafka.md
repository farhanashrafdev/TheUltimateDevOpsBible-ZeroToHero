# Message Queues & Apache Kafka: Complete Guide

## 🎯 Introduction

Message queues are fundamental to building scalable, resilient distributed systems. This guide covers message queue concepts, patterns, and deep dives into Apache Kafka and other popular messaging systems.

### Why Message Queues?

```
Without Message Queue:
┌─────────┐     Direct Call      ┌─────────┐
│Service A│ ─────────────────── │Service B│
└─────────┘   (Tight Coupling)   └─────────┘
     │                                │
     └── Service B down = Service A fails

With Message Queue:
┌─────────┐     ┌─────────┐     ┌─────────┐
│Service A│────▶│  Queue  │────▶│Service B│
└─────────┘     └─────────┘     └─────────┘
     │               │                │
     └── Service B down = Messages buffered
```

### Key Benefits

1. **Decoupling**: Services don't need to know about each other
2. **Asynchronous Processing**: Non-blocking operations
3. **Load Leveling**: Handle traffic spikes gracefully
4. **Reliability**: Messages persist until processed
5. **Scalability**: Add consumers as needed

## 📚 Message Queue Patterns

### Point-to-Point (Queue)

```
┌──────────┐     ┌─────────┐     ┌──────────┐
│ Producer │────▶│  Queue  │────▶│ Consumer │
└──────────┘     └─────────┘     └──────────┘

- One consumer processes each message
- Messages are removed after consumption
- Used for: task distribution, work queues
```

### Publish-Subscribe (Topic)

```
                              ┌──────────┐
                         ┌───▶│Consumer 1│
┌──────────┐   ┌───────┐ │    └──────────┘
│ Producer │──▶│ Topic │─┼───▶│Consumer 2│
└──────────┘   └───────┘ │    └──────────┘
                         └───▶│Consumer 3│
                              └──────────┘

- All consumers receive all messages
- Messages are broadcast
- Used for: event notification, logging
```

### Fan-Out Pattern

```
┌──────────┐     ┌─────────┐     ┌──────────┐
│ Producer │────▶│Exchange │──┬─▶│ Queue 1  │──▶ Email Service
└──────────┘     └─────────┘  │  └──────────┘
                              ├─▶│ Queue 2  │──▶ SMS Service
                              │  └──────────┘
                              └─▶│ Queue 3  │──▶ Push Service
                                 └──────────┘
```

### Request-Reply Pattern

```
┌──────────┐  Request   ┌─────────┐   ┌──────────┐
│ Requester│───────────▶│  Queue  │──▶│ Responder│
└──────────┘            └─────────┘   └──────────┘
      ▲                                     │
      │          ┌─────────┐               │
      └──────────│Reply Q  │◀──────────────┘
                 └─────────┘
```

## 🔥 Apache Kafka

### What is Kafka?

Apache Kafka is a distributed event streaming platform capable of handling trillions of events per day. Originally developed by LinkedIn, now open source under Apache.

### Kafka Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Kafka Cluster                             │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                  │
│  │ Broker 1 │  │ Broker 2 │  │ Broker 3 │                  │
│  │          │  │          │  │          │                  │
│  │ Topic A  │  │ Topic A  │  │ Topic A  │                  │
│  │ Part 0   │  │ Part 1   │  │ Part 2   │                  │
│  │ (Leader) │  │ (Leader) │  │ (Leader) │                  │
│  │          │  │          │  │          │                  │
│  │ Topic A  │  │ Topic A  │  │ Topic A  │                  │
│  │ Part 1   │  │ Part 2   │  │ Part 0   │                  │
│  │ (Replica)│  │ (Replica)│  │ (Replica)│                  │
│  └──────────┘  └──────────┘  └──────────┘                  │
│                                                              │
│  ┌────────────────────────────────────────────────────┐    │
│  │               ZooKeeper / KRaft                     │    │
│  │           (Cluster Coordination)                    │    │
│  └────────────────────────────────────────────────────┘    │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Key Concepts

```yaml
Topic:
  - Category/feed name for messages
  - Can have multiple partitions
  - Messages are ordered within partition

Partition:
  - Subset of topic for parallelism
  - Messages have offset (position)
  - Replicated across brokers

Producer:
  - Publishes messages to topics
  - Chooses partition (round-robin, key-based)
  - Can require acknowledgments

Consumer:
  - Reads messages from topics
  - Part of consumer group
  - Tracks offset (position)

Consumer Group:
  - Multiple consumers working together
  - Each partition assigned to one consumer
  - Enables parallel processing

Broker:
  - Kafka server
  - Stores messages
  - Handles requests

Offset:
  - Position of message in partition
  - Consumers track their offset
  - Enables replay and recovery
```

### Installation

#### Docker Compose

```yaml
# docker-compose.yml
version: '3'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.5.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - "2181:2181"

  kafka:
    image: confluentinc/cp-kafka:7.5.0
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1

  kafka-ui:
    image: provectuslabs/kafka-ui:latest
    ports:
      - "8080:8080"
    environment:
      KAFKA_CLUSTERS_0_NAME: local
      KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: kafka:9092
```

```bash
docker-compose up -d
```

#### Kubernetes (Strimzi)

```yaml
# kafka-cluster.yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: my-cluster
spec:
  kafka:
    version: 3.6.0
    replicas: 3
    listeners:
      - name: plain
        port: 9092
        type: internal
        tls: false
      - name: tls
        port: 9093
        type: internal
        tls: true
    config:
      offsets.topic.replication.factor: 3
      transaction.state.log.replication.factor: 3
      transaction.state.log.min.isr: 2
    storage:
      type: persistent-claim
      size: 100Gi
      class: standard
  zookeeper:
    replicas: 3
    storage:
      type: persistent-claim
      size: 10Gi
      class: standard
  entityOperator:
    topicOperator: {}
    userOperator: {}
```

### Kafka CLI Commands

```bash
# Create topic
kafka-topics.sh --create \
  --bootstrap-server localhost:9092 \
  --topic my-topic \
  --partitions 3 \
  --replication-factor 2

# List topics
kafka-topics.sh --list --bootstrap-server localhost:9092

# Describe topic
kafka-topics.sh --describe \
  --bootstrap-server localhost:9092 \
  --topic my-topic

# Produce messages
kafka-console-producer.sh \
  --bootstrap-server localhost:9092 \
  --topic my-topic

# Consume messages
kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic my-topic \
  --from-beginning

# Consumer with group
kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic my-topic \
  --group my-consumer-group

# List consumer groups
kafka-consumer-groups.sh --list \
  --bootstrap-server localhost:9092

# Describe consumer group
kafka-consumer-groups.sh --describe \
  --bootstrap-server localhost:9092 \
  --group my-consumer-group

# Reset offsets
kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --group my-consumer-group \
  --topic my-topic \
  --reset-offsets \
  --to-earliest \
  --execute
```

### Producer Code Examples

#### Python (kafka-python)

```python
from kafka import KafkaProducer
import json

# Create producer
producer = KafkaProducer(
    bootstrap_servers=['localhost:9092'],
    value_serializer=lambda v: json.dumps(v).encode('utf-8'),
    key_serializer=lambda k: k.encode('utf-8') if k else None,
    acks='all',  # Wait for all replicas
    retries=3,
    retry_backoff_ms=100
)

# Send message
def send_event(topic, key, value):
    future = producer.send(topic, key=key, value=value)
    # Block until sent (or timeout)
    record_metadata = future.get(timeout=10)
    print(f"Sent to {record_metadata.topic} partition {record_metadata.partition} offset {record_metadata.offset}")

# Example usage
event = {
    "user_id": "123",
    "action": "purchase",
    "amount": 99.99,
    "timestamp": "2024-01-15T10:30:00Z"
}
send_event("user-events", "user-123", event)

# Flush and close
producer.flush()
producer.close()
```

#### Python with Async

```python
from aiokafka import AIOKafkaProducer
import asyncio
import json

async def send_events():
    producer = AIOKafkaProducer(
        bootstrap_servers='localhost:9092',
        value_serializer=lambda v: json.dumps(v).encode('utf-8')
    )
    await producer.start()
    
    try:
        for i in range(100):
            event = {"event_id": i, "data": f"event-{i}"}
            await producer.send_and_wait("events", event)
    finally:
        await producer.stop()

asyncio.run(send_events())
```

#### Java

```java
import org.apache.kafka.clients.producer.*;
import java.util.Properties;

public class KafkaProducerExample {
    public static void main(String[] args) {
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092");
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("acks", "all");
        props.put("retries", 3);
        
        Producer<String, String> producer = new KafkaProducer<>(props);
        
        ProducerRecord<String, String> record = 
            new ProducerRecord<>("my-topic", "key", "value");
        
        producer.send(record, (metadata, exception) -> {
            if (exception == null) {
                System.out.printf("Sent to partition %d offset %d%n", 
                    metadata.partition(), metadata.offset());
            } else {
                exception.printStackTrace();
            }
        });
        
        producer.close();
    }
}
```

### Consumer Code Examples

#### Python

```python
from kafka import KafkaConsumer
import json

# Create consumer
consumer = KafkaConsumer(
    'user-events',
    bootstrap_servers=['localhost:9092'],
    group_id='my-consumer-group',
    auto_offset_reset='earliest',  # Start from beginning if no offset
    enable_auto_commit=True,
    auto_commit_interval_ms=5000,
    value_deserializer=lambda m: json.loads(m.decode('utf-8'))
)

# Process messages
for message in consumer:
    print(f"Topic: {message.topic}")
    print(f"Partition: {message.partition}")
    print(f"Offset: {message.offset}")
    print(f"Key: {message.key}")
    print(f"Value: {message.value}")
    print("---")

# Manual commit example
consumer = KafkaConsumer(
    'user-events',
    bootstrap_servers=['localhost:9092'],
    group_id='manual-commit-group',
    enable_auto_commit=False
)

for message in consumer:
    try:
        process_message(message)
        consumer.commit()  # Commit after successful processing
    except Exception as e:
        print(f"Error processing message: {e}")
        # Don't commit - message will be reprocessed
```

#### Python Batch Processing

```python
from kafka import KafkaConsumer
import json

consumer = KafkaConsumer(
    'user-events',
    bootstrap_servers=['localhost:9092'],
    group_id='batch-processor',
    enable_auto_commit=False,
    max_poll_records=100  # Batch size
)

while True:
    # Poll for batch of messages
    messages = consumer.poll(timeout_ms=1000, max_records=100)
    
    for topic_partition, records in messages.items():
        batch = [json.loads(r.value) for r in records]
        
        try:
            # Process batch
            process_batch(batch)
            # Commit offset after batch
            consumer.commit()
        except Exception as e:
            print(f"Batch processing failed: {e}")
```

### Kafka Streams

```java
import org.apache.kafka.streams.*;
import org.apache.kafka.streams.kstream.*;

public class WordCountStream {
    public static void main(String[] args) {
        Properties props = new Properties();
        props.put(StreamsConfig.APPLICATION_ID_CONFIG, "wordcount-app");
        props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        
        StreamsBuilder builder = new StreamsBuilder();
        
        KStream<String, String> textLines = builder.stream("text-input");
        
        KTable<String, Long> wordCounts = textLines
            .flatMapValues(line -> Arrays.asList(line.toLowerCase().split("\\W+")))
            .groupBy((key, word) -> word)
            .count();
        
        wordCounts.toStream().to("word-counts");
        
        KafkaStreams streams = new KafkaStreams(builder.build(), props);
        streams.start();
    }
}
```

### Kafka Configuration Best Practices

```yaml
# Producer Configuration
producer:
  acks: all                           # Strongest durability
  retries: 2147483647                 # Max retries
  max.in.flight.requests.per.connection: 5  # With idempotence
  enable.idempotence: true           # Exactly-once semantics
  compression.type: lz4              # Good balance of speed/compression
  batch.size: 16384                  # 16KB batches
  linger.ms: 5                       # Wait 5ms for more messages
  buffer.memory: 33554432            # 32MB buffer

# Consumer Configuration
consumer:
  group.id: my-consumer-group
  auto.offset.reset: earliest        # Or 'latest'
  enable.auto.commit: false          # Manual commit for reliability
  max.poll.records: 500              # Batch size
  max.poll.interval.ms: 300000       # 5 min max processing time
  session.timeout.ms: 45000          # Consumer heartbeat timeout
  heartbeat.interval.ms: 15000       # Heartbeat frequency

# Broker Configuration
broker:
  num.partitions: 6                  # Default partitions
  default.replication.factor: 3      # Replicas per partition
  min.insync.replicas: 2            # Min replicas for write
  log.retention.hours: 168          # 7 days retention
  log.retention.bytes: -1           # No size limit
  log.segment.bytes: 1073741824     # 1GB segments
```

## 🐰 RabbitMQ

### Overview

RabbitMQ is a traditional message broker implementing AMQP (Advanced Message Queuing Protocol).

### When to Use RabbitMQ vs Kafka

```yaml
Use RabbitMQ when:
  - Need complex routing (headers, topics)
  - Priority queues required
  - Message acknowledgment important
  - Smaller scale (<100K msg/sec)
  - Traditional queue patterns

Use Kafka when:
  - Very high throughput needed
  - Event streaming/replay required
  - Log aggregation use case
  - Need message persistence
  - Real-time analytics
```

### RabbitMQ Docker Setup

```yaml
# docker-compose.yml
version: '3'
services:
  rabbitmq:
    image: rabbitmq:3-management
    ports:
      - "5672:5672"    # AMQP
      - "15672:15672"  # Management UI
    environment:
      RABBITMQ_DEFAULT_USER: admin
      RABBITMQ_DEFAULT_PASS: admin
```

### RabbitMQ Python Example

```python
import pika
import json

# Connection
connection = pika.BlockingConnection(
    pika.ConnectionParameters('localhost')
)
channel = connection.channel()

# Declare queue
channel.queue_declare(queue='task_queue', durable=True)

# Publish message
message = {"task": "process_order", "order_id": 123}
channel.basic_publish(
    exchange='',
    routing_key='task_queue',
    body=json.dumps(message),
    properties=pika.BasicProperties(
        delivery_mode=2,  # Make message persistent
    )
)

# Consumer
def callback(ch, method, properties, body):
    message = json.loads(body)
    print(f"Received: {message}")
    # Process message
    ch.basic_ack(delivery_tag=method.delivery_tag)

channel.basic_qos(prefetch_count=1)
channel.basic_consume(queue='task_queue', on_message_callback=callback)
channel.start_consuming()
```

## ☁️ AWS SQS & SNS

### SQS (Simple Queue Service)

```python
import boto3
import json

sqs = boto3.client('sqs', region_name='us-east-1')
queue_url = 'https://sqs.us-east-1.amazonaws.com/123456789/my-queue'

# Send message
response = sqs.send_message(
    QueueUrl=queue_url,
    MessageBody=json.dumps({'order_id': 123}),
    MessageAttributes={
        'OrderType': {
            'StringValue': 'standard',
            'DataType': 'String'
        }
    }
)

# Receive messages
response = sqs.receive_message(
    QueueUrl=queue_url,
    MaxNumberOfMessages=10,
    WaitTimeSeconds=20,  # Long polling
    MessageAttributeNames=['All']
)

for message in response.get('Messages', []):
    print(message['Body'])
    # Process message
    # Delete after processing
    sqs.delete_message(
        QueueUrl=queue_url,
        ReceiptHandle=message['ReceiptHandle']
    )
```

### SNS (Simple Notification Service)

```python
import boto3

sns = boto3.client('sns', region_name='us-east-1')
topic_arn = 'arn:aws:sns:us-east-1:123456789:my-topic'

# Publish to topic
response = sns.publish(
    TopicArn=topic_arn,
    Message=json.dumps({'event': 'user_signup', 'user_id': 123}),
    Subject='User Event'
)

# Subscribe SQS queue to SNS topic
sns.subscribe(
    TopicArn=topic_arn,
    Protocol='sqs',
    Endpoint='arn:aws:sqs:us-east-1:123456789:my-queue'
)
```

## 🏗️ Event-Driven Architecture Patterns

### Event Sourcing

```
┌─────────────────────────────────────────────────────────────┐
│                    Event Sourcing                            │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Instead of storing current state, store all events:        │
│                                                              │
│  Event Store:                                                │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ OrderCreated {id:1, items:[...], total:100}         │   │
│  │ PaymentReceived {order_id:1, amount:100}            │   │
│  │ OrderShipped {order_id:1, tracking:"ABC123"}        │   │
│  │ OrderDelivered {order_id:1, date:"2024-01-15"}      │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                              │
│  Benefits:                                                   │
│  ├── Complete audit trail                                   │
│  ├── Temporal queries (state at any time)                  │
│  ├── Event replay for debugging                            │
│  └── Easy to build read models (CQRS)                      │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### CQRS (Command Query Responsibility Segregation)

```
┌─────────────────────────────────────────────────────────────┐
│                         CQRS                                 │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Commands (Write)              Queries (Read)               │
│       │                              │                       │
│       ▼                              ▼                       │
│  ┌─────────┐                   ┌─────────┐                  │
│  │ Command │                   │  Query  │                  │
│  │ Handler │                   │ Handler │                  │
│  └────┬────┘                   └────┬────┘                  │
│       │                              │                       │
│       ▼                              ▼                       │
│  ┌─────────┐    Events        ┌─────────┐                  │
│  │ Event   │ ───────────────▶ │  Read   │                  │
│  │ Store   │                   │  Model  │                  │
│  └─────────┘                   └─────────┘                  │
│                                                              │
│  Benefits:                                                   │
│  ├── Optimized read models                                  │
│  ├── Independent scaling                                    │
│  └── Different storage technologies                        │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Saga Pattern (Distributed Transactions)

```
┌─────────────────────────────────────────────────────────────┐
│                    Saga Pattern                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Order Saga:                                                 │
│                                                              │
│  ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐    │
│  │ Create  │──▶│ Reserve │──▶│ Process │──▶│  Ship   │    │
│  │ Order   │   │Inventory│   │ Payment │   │  Order  │    │
│  └─────────┘   └─────────┘   └─────────┘   └─────────┘    │
│       │             │             │             │          │
│       │  Compensate │  Compensate │  Compensate │          │
│       ▼             ▼             ▼             ▼          │
│  ┌─────────┐   ┌─────────┐   ┌─────────┐                  │
│  │ Cancel  │◀──│ Release │◀──│ Refund  │                  │
│  │ Order   │   │Inventory│   │ Payment │                  │
│  └─────────┘   └─────────┘   └─────────┘                  │
│                                                              │
│  Choreography: Services react to events                     │
│  Orchestration: Central coordinator manages flow           │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## 📊 Monitoring Kafka

### Key Metrics

```yaml
Broker Metrics:
  - kafka.server:type=BrokerTopicMetrics,name=MessagesInPerSec
  - kafka.server:type=BrokerTopicMetrics,name=BytesInPerSec
  - kafka.server:type=BrokerTopicMetrics,name=BytesOutPerSec
  - kafka.server:type=ReplicaManager,name=UnderReplicatedPartitions
  - kafka.server:type=ReplicaManager,name=PartitionCount

Consumer Metrics:
  - kafka.consumer:type=consumer-fetch-manager-metrics,client-id=*,attribute=records-lag-max
  - kafka.consumer:type=consumer-coordinator-metrics,client-id=*,attribute=commit-latency-avg

Producer Metrics:
  - kafka.producer:type=producer-metrics,client-id=*,attribute=record-send-rate
  - kafka.producer:type=producer-metrics,client-id=*,attribute=record-error-rate
```

### Prometheus + Grafana Setup

```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'kafka'
    static_configs:
      - targets: ['kafka:9308']  # JMX Exporter
```

### Kafka Lag Monitoring

```python
from kafka import KafkaConsumer, KafkaAdminClient
from kafka.admin import ConsumerGroupDescription

admin = KafkaAdminClient(bootstrap_servers='localhost:9092')

# Get consumer group lag
def get_consumer_lag(group_id):
    consumer = KafkaConsumer(
        bootstrap_servers='localhost:9092',
        group_id=group_id
    )
    
    # Get committed offsets
    committed = consumer.committed(consumer.assignment())
    
    # Get end offsets
    end_offsets = consumer.end_offsets(consumer.assignment())
    
    total_lag = 0
    for tp in consumer.assignment():
        lag = end_offsets[tp] - (committed[tp] or 0)
        total_lag += lag
        print(f"Partition {tp.partition}: lag = {lag}")
    
    return total_lag
```

## ✅ Best Practices

### Message Design

```yaml
Do:
  - Use schema registry (Avro, Protobuf)
  - Include message metadata (timestamp, source)
  - Design for idempotency
  - Version your messages
  - Keep messages small (<1MB)

Don't:
  - Store large blobs in messages
  - Rely on message ordering across partitions
  - Use topics as databases
  - Ignore dead letter queues
```

### Partition Strategy

```yaml
Partitioning:
  - Use meaningful keys (user_id, order_id)
  - More partitions = more parallelism
  - Partition count is hard to change
  - Start with: 2-3x expected consumer count

Key Selection:
  - Same key = same partition = ordering guarantee
  - Null key = round-robin distribution
  - Avoid hot partitions (celebrity problem)
```

### Error Handling

```python
# Dead Letter Queue Pattern
def process_with_dlq(message):
    try:
        process_message(message)
    except Exception as e:
        # Send to DLQ after max retries
        if message.retry_count >= MAX_RETRIES:
            send_to_dlq(message, error=str(e))
        else:
            # Requeue with incremented retry count
            requeue_with_delay(message)
```

## ✅ Mastery Checklist

- [ ] Understand message queue patterns
- [ ] Set up Kafka cluster
- [ ] Create producers and consumers
- [ ] Implement consumer groups
- [ ] Handle errors and retries
- [ ] Monitor consumer lag
- [ ] Implement exactly-once semantics
- [ ] Design event-driven architectures
- [ ] Use schema registry
- [ ] Implement dead letter queues

---

**Next Steps**:
- Learn [Monitoring & Observability](./monitoring-observability.md)
- Explore [Service Mesh](./service-mesh.md)
- Master [Microservices Architecture](./containers.md)

**Remember**: Message queues are the backbone of distributed systems. Start with simple patterns, understand the guarantees, and choose the right tool for your use case. Kafka for high-throughput streaming, RabbitMQ for complex routing.


