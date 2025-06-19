## Lesson 2:  Introduction to Kafka

### What is Kafka?

Apache Kafka is an open-source distributed streaming platform that enables you to publish, subscribe to, store, and process streams of records in a fault-tolerant way. It's often described as a distributed commit log or a distributed publish-subscribe messaging system. Unlike traditional messaging systems, Kafka is designed for high-throughput, low-latency data ingestion, and processing, making it ideal for real-time data pipelines and streaming applications.

### Brief History of Kafka

Kafka was originally developed at LinkedIn in 2011 to handle the company's massive volume of activity stream data. It was designed to provide a high-throughput, low-latency platform for handling event streams at scale. In 2012, it was open-sourced under the Apache Software Foundation, and since then, it has grown into a vibrant open-source community with widespread adoption across various industries.

### Why Kafka? (Benefits and Use Cases )

Kafka's popularity stems from its ability to address common challenges in data integration and processing. Here are some key benefits and typical use cases:

**Benefits:**
<p align="center">
  <img src="L2_Assests/Benefits of Kafka..png" alt="Benefits of Kafka" width="80%" />
</p>

**Use Cases:**

  
| **Use Case**         | **Description**                                                                                                   | **Real-World Example**                                                                                                                 |
|----------------------|-------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------|
| Real-time Analytics  | Ingesting and processing sensor data, website activity, or financial transactions for immediate insights.         | Pinterest uses Kafka to collect real-time user interaction data (pins, clicks, impressions) and feed into real-time analytics dashboards.           |
| Log Aggregation      | Centralising logs from various applications and servers for monitoring and analysis.                              | Netflix uses Kafka to aggregate logs from its microservices architecture to monitor streaming quality and infrastructure health.       |
| Event Sourcing       | Storing all changes to an application's state as a sequence of immutable events.                                 | LinkedIn uses Kafka to track user interactions and metrics as append-only events across messaging and recommendation services.         |
| Stream Processing    | Building applications that continuously process streams of data (e.g., fraud detection, anomaly detection).       | Grab leverages Kafka Streams with ML models to power GrabDefence, a fraud detection system processing billions of events daily.        |
| Message Queues       | A high-throughput alternative to traditional message queues for decoupling applications.                          | Airbnb uses Kafka as a replacement for traditional messaging queues to enable real-time communication between services.|
| Data Integration     | Connecting various systems by acting as a central hub for data exchange.                                          | ING Bank uses Kafka as a central nervous system to connect legacy and modern systems, enabling real-time data movement across domains. |


-----

## 2. Kafka Core Concepts & Architecture

Understanding Kafka's core concepts is crucial for effective utilization.

### Distributed Commit Log

At its heart, Kafka can be thought of as a **distributed commit log**. This means it appends records (messages) to a log in an ordered, immutable fashion. This log is distributed across multiple servers (brokers) for scalability and fault tolerance. Each record in the log is assigned a unique, sequential ID called an **offset**.

### Topics, Partitions, and Offsets

  * **Topic:** A named logical channel to which records are published. Think of a topic as a category or feed name where messages of a particular type are sent. For example, you might have a topic for "user_signups" or "payment_transactions."

  * **Partition:** A topic is divided into one or more partitions. Each partition is an ordered, immutable sequence of records. Records within a partition are strictly ordered, but there is no guaranteed order across different partitions of the same topic. Partitions are the unit of parallelism in Kafka. More partitions allow for higher throughput.

  * **Offset:** Each record within a partition is assigned a unique, sequential ID called an offset. This offset acts as a pointer to a specific record within a partition. Consumers use offsets to track their position within a partition, ensuring they consume messages in the correct order and don't miss any.

    ```
    Topic A
    ├── Partition 0 [Record 0, Record 1, Record 2, ...]
    └── Partition 1 [Record 0, Record 1, Record 2, ...]
    ```

Before diving into the individual components of the Kafka ecosystem, here's a high-level visual overview that illustrates how these core parts—ZooKeeper, Producers, Brokers, and Consumers—interact within the system.

<p align="center">
  <img src="L2_Assests/Kafka System Cycle..png" alt="Kafka System Overview" width="60%" />
</p>

### Producers

**Producers** are client applications that publish (write) records to Kafka topics. When a producer sends a record, it specifies the topic and can optionally provide a key. The key is used to determine which partition the record will be written to (records with the same key are guaranteed to go to the same partition). Producers can send records synchronously or asynchronously.

### Consumers and Consumer Groups

  * **Consumers:** Client applications that subscribe to (read) records from Kafka topics. Consumers read data from one or more partitions of a topic.

  * **Consumer Group:** A group of consumers that work together to consume messages from a topic. Each partition within a topic is assigned to only one consumer within a consumer group at any given time. This ensures that messages are processed by only one consumer in the group, providing load balancing and fault tolerance. If a consumer in the group fails, its partitions are automatically reassigned to other consumers in the same group.

    ```
    Topic A (3 Partitions)
    ├── Partition 0 -> Consumer Group X (Consumer 1)
    ├── Partition 1 -> Consumer Group X (Consumer 2)
    └── Partition 2 -> Consumer Group X (Consumer 3)
    ```

### Brokers

**Brokers** are the Kafka servers that form the Kafka cluster. Each broker stores a subset of the topic partitions and handles requests from producers and consumers. Brokers are responsible for:

  * Receiving messages from producers.
  * Storing messages on disk.
  * Serving messages to consumers.
  * Replicating partitions for fault tolerance.

A Kafka cluster consists of one or more brokers.

### ZooKeeper 

While newer versions of Kafka are moving towards removing the dependency, historically and for most existing deployments, **Apache ZooKeeper** plays a critical role in a Kafka cluster. ZooKeeper is a distributed coordination service that Kafka uses for:

  * **Broker Discovery:** Brokers register themselves with ZooKeeper, allowing producers and consumers to discover available brokers.
  * **Controller Election:** One broker in the cluster is elected as the "controller" by ZooKeeper. The controller is responsible for administrative tasks like assigning partitions to brokers and handling broker failures.
  * **Topic Configuration:** Storing metadata about topics, partitions, and their replicas.
  * **Consumer Group Offsets (older clients):** Historically, consumer offsets were stored in ZooKeeper. Modern Kafka clients (new consumer API) store offsets directly in a special Kafka topic (`__consumer_offsets`).

### Replication and Durability

Kafka ensures **durability** and **fault tolerance** through replication. Each partition can have multiple replicas. One replica is designated as the **leader**, and the others are **followers**.

  * **Leader:** All read and write operations for a partition go through its leader.
  * **Followers:** Followers passively replicate the leader's data. If the leader fails, one of the followers is automatically elected as the new leader, ensuring continuous availability of the data.

The number of replicas for a topic is called its **replication factor**. A replication factor of 3 means there is one leader and two followers for each partition. This provides resilience against up to two broker failures without data loss.

-----

## 3. Kafka Tools & Integrations

Beyond the core messaging architecture, Kafka supports several powerful tools and APIs that enhance its functionality. These are essential when building real-time, scalable, and production-ready data solutions.

<p align="center">
  <img src="L2_Assests/Kafka Tools and Integrations..png" alt="Kafka Tools & Integrations at a Glance" width="80%" />
</p>

### Kafka Connect

**Kafka Connect** is a framework for building and running scalable, reliable data pipelines. It simplifies the process of integrating Kafka with other systems (e.g., databases, file systems, message queues). It offers two types of connectors:

- **Source Connectors:** Ingest data from external systems into Kafka topics.
- **Sink Connectors:** Deliver data from Kafka topics to external systems.

Kafka Connect handles much of the boilerplate code involved in data integration, including fault tolerance, scalability, and offset management.

### Kafka Streams

**Kafka Streams** is a client-side library for building real-time stream processing applications. It enables developers to transform, filter, aggregate, and join data streams directly within their application code. Key features include:

- **Low Latency:** Processes data as it arrives with minimal delay.
- **Fault Tolerance:** Built-in resilience and state management.
- **Scalability:** Uses Kafka's consumer group model for horizontal scaling.
- **Exactly-once Processing:** Ensures each record is processed exactly once, even during failures.

Kafka Streams eliminates the need for separate distributed processing frameworks, making stream logic easier to integrate directly into applications.

### Kafka Clients (APIs)

Kafka provides official client libraries (APIs) in various programming languages to enable applications to interact with the cluster:

- **Java Client:** The primary and most mature client.
- **Python Client:** Libraries such as `kafka-python` and `confluent-kafka-python`.
- **Go Client:** Libraries like `confluent-kafka-go`.
- **Node.js Client:** Libraries such as `kafkajs`.
- **Others:** Community-supported clients for languages like C#, Rust, and more.

These clients abstract Kafka’s communication protocol, allowing developers to work in the language of their tech stack without worrying about underlying complexities.

### Schema Registry (Confluent Platform)

Although not part of the core Apache Kafka project, the **Schema Registry**—available via the Confluent Platform—is widely used in production deployments. It provides a central repository for managing schemas (i.e., the structure of Kafka messages) and offers several critical features:

- **Schema Evolution:** Supports backward and forward compatibility to allow for safe schema updates.
- **Data Governance:** Enforces consistent data formats across systems for better quality control.
- **Serialization/Deserialization:** Integrates with formats like Avro, Protobuf, and JSON Schema to automate data encoding and decoding.

The Schema Registry is essential for teams managing structured data, ensuring safe and consistent data exchange between services.

## 4. Kafka Setup & CLI

Now that you’ve learned about Kafka’s architecture and supporting tools, let’s walk through setting up Kafka locally and running it using the command line.

This hands-on section is ideal for development, experimentation, and understanding how Kafka works behind the scenes.

---

### Kafka Setup Workflow (Visual Overview)

Here’s a step-by-step view of the full setup process:

<p align="center">
  <img src="L2_Assests/Kafka Setup and Usage Flowchart..png" alt="Kafka Setup and Usage Flowchart" width="290px" />
</p>

### Prerequisites (Java)

Kafka is written in Scala and runs on the Java Virtual Machine (JVM). Therefore, Java must be installed on your system.

- **Java Development Kit (JDK 8 or higher)** is generally recommended.

You can verify your Java installation with:

```bash
java -version
```

---

### Downloading and Extracting Kafka

1. **Download:** [Visit the official Apache Kafka website](https://kafka.apache.org/downloads)

   Choose the latest stable release (e.g., `kafka_2.13-3.x.x.tgz`).  
   The `2.13` refers to the Scala version used.

3. **Extract the archive:**

```bash
tar -xzf kafka_2.13-3.x.x.tgz
cd kafka_2.13-3.x.x
```

Let’s assume you're in the extracted Kafka directory for the next steps.

---

### A. Starting ZooKeeper

Kafka traditionally uses **ZooKeeper** for coordination tasks. You’ll need to start it before launching Kafka.

```bash
bin/zookeeper-server-start.sh config/zookeeper.properties
```

This will start ZooKeeper and stream logs in the terminal. Leave this window open.

---

### B. Starting Kafka Broker

Now open a **new terminal window** and start the Kafka broker.

```bash
bin/kafka-server-start.sh config/server.properties
```

Once you see a message like `[KafkaServer id=0] started`, the broker is running successfully.

---

### C. Basic CLI Commands

Let’s explore how to work with Kafka using its command-line interface.

---

#### Creating Topics

Before producing or consuming messages, you need a topic.

```bash
bin/kafka-topics.sh --create --topic my_first_topic --bootstrap-server localhost:9092 --partitions 3 --replication-factor 1
```

For single-broker setups, keep the replication factor as `1`.

---

#### Listing Topics

To confirm your topic was created:

```bash
bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```

You should see `my_first_topic` listed.

---

#### Producing Messages

Use the Kafka console producer to send messages to the topic.

```bash
bin/kafka-console-producer.sh --topic my_first_topic --bootstrap-server localhost:9092
```

Type your messages and press Enter after each:

```
>Hello Kafka!
>This is my first message.
>Learning Kafka is fun!
```

Press `Ctrl+C` to stop the producer.

---

#### Consuming Messages

To read messages from the topic, use the Kafka console consumer.

```bash
bin/kafka-console-consumer.sh --topic my_first_topic --bootstrap-server localhost:9092 --from-beginning
```

Expected output:

```
Hello Kafka!
This is my first message.
Learning Kafka is fun!
```

Keep this window open to see new messages in real-time. Press `Ctrl+C` to exit.

---

#### Deleting Topics

You can delete a topic after enabling deletion in the Kafka config:

1. Open `config/server.properties`  
2. Set: `delete.topic.enable=true`  
3. Restart the Kafka broker

Then run:

```bash
bin/kafka-topics.sh --delete --topic my_first_topic --bootstrap-server localhost:9092
```

Kafka will mark the topic for deletion.

---

### Kafka CLI Command Cheat Sheet

<div align="center">

| **Command** | **Purpose** |
|-------------|-------------|
| `java -version` | Verify Java installation |
| `tar -xzf kafka_*.tgz` | Extract Kafka archive |
| `zookeeper-server-start.sh` | Start ZooKeeper |
| `kafka-server-start.sh` | Start Kafka broker |
| `kafka-topics.sh --create` | Create a Kafka topic |
| `kafka-topics.sh --list` | List available topics |
| `kafka-console-producer.sh` | Send messages to a topic |
| `kafka-console-consumer.sh` | Read messages from a topic |
| `kafka-topics.sh --delete` | Delete a topic |

</div>

---
## 5. Kafka with Python

Now that you've interacted with Kafka using the command line, let's take the next step: integrating Kafka into a Python application.

Python is widely used in data engineering, streaming pipelines, and backend development—making it a popular choice for working with Kafka. We'll use the `kafka-python` library for the examples below.

<p align="center">
  <img src="L2_Assests/Kafka Message Flow with Python..png" alt="Kafka Message Flow with Python" width="100%" />
</p>

---

### Setting up the Python Environment

It's best practice to use a virtual environment for your Python projects:

```bash
python3 -m venv kafka_env
source kafka_env/bin/activate  # On Windows: .\kafka_env\Scripts\activate
```

---

### Installing `kafka-python`

Use pip to install the Kafka client library:

```bash
pip install kafka-python
```

---

### Basic Producer Example

*Use Case:* This script acts as a producer application that publishes events (e.g., sensor readings) to Kafka.

Create a file named `producer.py`:

```python
from kafka import KafkaProducer
import json
import time

# Create a KafkaProducer instance
producer = KafkaProducer(
    bootstrap_servers='localhost:9092',
    value_serializer=lambda v: json.dumps(v).encode('utf-8')  # Convert dict to JSON bytes
)

topic_name = 'python_topic'
print(f"Producing messages to topic: {topic_name}")

for i in range(5):
    message = {'number': i, 'timestamp': time.time()}
    print(f"Sending: {message}")
    producer.send(topic_name, message)
    time.sleep(1)

producer.flush()  # Ensure all messages are sent
print("Messages sent successfully!")
producer.close()
```

Run the producer:

```bash
python producer.py
```

---

### Basic Consumer Example

*Use Case:*  This script acts as a consumer service that listens to a Kafka topic and processes messages.

Create a file named `consumer.py`:

```python
from kafka import KafkaConsumer
import json

topic_name = 'python_topic'

consumer = KafkaConsumer(
    topic_name,
    bootstrap_servers='localhost:9092',
    auto_offset_reset='earliest',  # Start from the beginning if no offset is found
    enable_auto_commit=True,       # Automatically commit read offsets
    group_id='my_python_group',    # Consumer group ID
    value_deserializer=lambda x: json.loads(x.decode('utf-8'))  # Convert bytes to dict
)

print(f"Consuming messages from topic: {topic_name}")

try:
    for message in consumer:
        print(f"Received message: Partition={message.partition}, Offset={message.offset}, Value={message.value}")
except KeyboardInterrupt:
    print("Consumer stopped by user.")
finally:
    consumer.close()
```

Before running the consumer, make sure you've created the topic:

```bash
bin/kafka-topics.sh --create --topic python_topic --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
```

Run the consumer:

```bash
python consumer.py
```

---

### Working with Consumer Groups in Python

The `consumer.py` script already includes a consumer group ID: `my_python_group`.

- If you run **multiple consumers with the same `group_id`**, they will form a **consumer group** and split the partitions between them.
- If you use **different `group_id`s**, each consumer will read **all messages independently**, even if they read from the same topic.

---

### Handling Serialization / Deserialization

Kafka messages are sent and received as **byte arrays**. To work with structured data (e.g., JSON), we use custom serializer and deserializer functions:

- `value_serializer` in `KafkaProducer`: Converts a Python object (e.g., dict) to bytes.
- `value_deserializer` in `KafkaConsumer`: Converts bytes back to a Python object.

In our examples:
```python
value_serializer=lambda v: json.dumps(v).encode('utf-8')
value_deserializer=lambda x: json.loads(x.decode('utf-8'))
```

This makes it easy to send structured JSON data over Kafka.

For production setups with stricter data governance, consider using **Avro or Protobuf** with a **Schema Registry** and libraries like `confluent-kafka-python`.

---

### Kafka with Python – Quick Reference Table

<div align="center">

| **Component**        | **Purpose**                                 | **Used In**     |
|----------------------|---------------------------------------------|-----------------|
| `KafkaProducer`      | Sends messages to Kafka                     | `producer.py`   |
| `KafkaConsumer`      | Reads messages from Kafka                   | `consumer.py`   |
| `value_serializer`   | Converts dict → JSON → bytes                | `producer.py`   |
| `value_deserializer` | Converts bytes → JSON → dict                | `consumer.py`   |
| `group_id`           | Enables consumer group coordination         | `consumer.py`   |

</div>

## 6. Security, Monitoring & Tuning

Once you move beyond local development, these aspects become critical for a production-ready Kafka cluster. In real-world deployments, securing your Kafka ecosystem, monitoring its performance, and fine-tuning its behaviour are essential for reliability, efficiency, and data integrity.

---

### Security

Kafka often operates as a central nervous system for data exchange. As such, security is not optional—it’s vital. Kafka security focuses on three areas: **Authentication**, **Authorization**, and **Encryption**. Together, they ensure that only the right systems access the right data securely.

<div align="center">

| **Aspect**        | **Purpose**                                                                 | **Common Methods**                                                   |
|-------------------|------------------------------------------------------------------------------|-----------------------------------------------------------------------|
| **Authentication**| Verifies the identity of clients (producers/consumers) and brokers          | SASL/PLAIN, SASL/SCRAM, SASL/GSSAPI (Kerberos), SASL/OAUTHBEARER     |
| **Authorization** | Controls what authenticated clients are allowed to do (e.g., topic access)  | Kafka Access Control Lists (ACLs)                                    |
| **Encryption**    | Encrypts data in transit between clients and brokers to prevent tampering   | SSL/TLS                                                               |

</div>

You apply these security layers through Kafka’s configuration files:
- `server.properties` on brokers (for cluster-level security)  
- `producer.properties` and `consumer.properties` on clients (to authenticate and encrypt communication)

> In practice, security setup varies by organisation. Some use OAuth2 for token-based auth; others rely on Kerberos for enterprise-grade integration.

---

### Monitoring

Kafka can process millions of messages per second, but that power means it must be observed constantly. Monitoring ensures your system remains stable, responsive, and fault-tolerant under load.

#### Native Monitoring Tools

Kafka provides its own mechanisms for performance tracking:

- **JMX Metrics:** Java-based metrics exposed for system introspection (latency, throughput, lag, etc.)
- **Kafka Manager (Yahoo!):** Lightweight open-source tool for monitoring cluster status and topic metrics
- **Confluent Control Center:** Full-featured dashboard with alerts, stream governance, and performance insights

#### External Monitoring Solutions

Many production teams integrate Kafka into broader observability stacks:

- **Prometheus + Grafana:** Open-source combo for scraping JMX metrics and visualising them in real-time
- **Datadog, New Relic, Splunk:** Commercial tools with Kafka plugins, often used in enterprise-grade observability pipelines

> Choosing your monitoring setup depends on your environment—open-source teams often prefer Prometheus, while enterprises favour tools like Datadog for SLA enforcement.

<div align="center">

| **Metric**                  | **What It Tells You**                                               |
|-----------------------------|----------------------------------------------------------------------|
| **Broker Health**           | Tracks CPU, memory, disk usage, and network I/O                     |
| **Topic Throughput**        | Measures message volume per topic (bytes in/out, messages per sec)  |
| **Consumer Lag**            | Shows if consumers are falling behind in message processing         |
| **Under-Replicated Partitions** | Flags potential availability or redundancy issues                  |

</div>

---

### Tuning

Kafka is highly configurable, which makes it powerful—but tuning those configurations is key to ensuring efficiency at scale. Tuning affects **throughput, latency, durability, and fault tolerance**—so adjustments should align with your performance goals.

#### Broker Configurations (`server.properties`)

Broker settings affect how Kafka handles data persistence, networking, and replication:

- `num.partitions`: Determines parallelism across topics
- `log.retention.hours` / `log.retention.bytes`: Sets how long or how much data is stored
- `default.replication.factor`: Number of brokers holding a copy of each partition
- `num.network.threads`, `num.io.threads`: Controls internal thread pools for request handling
- `log.segment.bytes`, `log.segment.ms`: When to roll over Kafka logs (based on size/time)

#### Producer Configurations

These settings affect how producers buffer and send data to Kafka:

- `acks`: Controls data durability guarantees (`0`, `1`, or `"all"`)
- `batch.size`: Batches small messages together for efficient transmission
- `linger.ms`: Adds delay to wait for batch accumulation before send
- `compression.type`: Reduces message size for better throughput (e.g., gzip, lz4)

> Use `acks=all` for critical data pipelines where message loss is unacceptable.

#### Consumer Configurations

Consumers must balance throughput with responsiveness:

- `fetch.min.bytes`: Don’t respond to fetch until this many bytes are available
- `fetch.max.wait.ms`: Maximum time the broker will wait to respond
- `session.timeout.ms`: Time after which a consumer is considered inactive
- `max.poll.records`: Cap on records returned in a single poll
- `auto.offset.reset`: Action to take if no valid offset is found (`earliest`, `latest`, `none`)

> Setting a low `max.poll.records` with high throughput topics can lead to lag.

#### Operating System Tuning

Kafka is disk- and memory-intensive. OS-level tuning can significantly affect performance:

- **File Descriptors:** Set `ulimit -n` high enough to support topic partitions
- **Disk I/O:** Prefer SSDs or RAID arrays for persistence performance
- **Network:** Increase socket buffers to handle high message volumes
- **Memory:** Allocate enough heap and off-heap memory to JVM (e.g., via `KAFKA_HEAP_OPTS`)

---

> Tuning is an **iterative process**. Always benchmark changes in a staging environment before deploying to production.

---

## 7. Case Studies

Kafka's versatility has led to widespread adoption across industries—from tech giants to banks—serving as the backbone for real-time, resilient data pipelines. Here are some notable examples that demonstrate its scalability, performance, and flexibility.

---

### LinkedIn (Kafka's Origin Story)

**Why Kafka was created:**  
LinkedIn developed Kafka to handle massive volumes of user activity data.

**Use Cases:**
- **Activity Stream Processing:** Powering real-time news feeds, recommendations, and analytics
- **Operational Monitoring:** Collecting server metrics for dashboards and alerting
- **Internal System Integration:** Connecting internal apps and databases with high throughput

---

### Netflix

**Why Kafka:**  
Netflix uses Kafka to support its complex, microservices-based architecture and high availability demands.

**Use Cases:**
- **Real-time Monitoring:** Infrastructure telemetry for anomaly detection
- **Playback Tracking:** Events used for personalisation, billing, and engagement insights
- **Log Aggregation:** Unified logging pipeline for all microservices
- **Data Pipelining:** Moving data between services, warehouses, and ML models

---

### Uber

**Why Kafka:**  
Uber relies on Kafka for its dynamic, data-driven operations at global scale.

**Use Cases:**
- **Real-time Ride Matching:** Streaming driver and rider location updates
- **Fraud Detection:** Analysing financial and behavioural data in real time
- **Surge Pricing:** Adjusting pricing based on supply-demand telemetry
- **Operational Intelligence:** Live insights into trips, driver availability, delays, etc.


---

## 8. Conclusion

These real-world case studies showcase how Kafka enables organisations to build systems that are:

- **Scalable** (from startups to tech giants)
- **Reliable** (fault-tolerant under pressure)
- **Real-time** (powering decisions as data flows)

Whether you're building dashboards, alerting systems, fraud detection engines, or recommender systems, Kafka is likely under the hood—making things move in milliseconds.

  * **Recommendation Engines:** Feeding user behavior data into real-time recommendation systems.
  * **IoT Data Ingestion:** Collecting massive volumes of data from IoT devices for monitoring, predictive maintenance, and operational optimization.

These case studies highlight Kafka's flexibility and power as a central nervous system for data, enabling organizations to build highly scalable, resilient, and real-time data-driven applications.

---

## 9. Activities

### Activity 1. Real-Time Temperature Sensor Stream

**Objective:**
Build a real-time temperature monitoring system using Apache Kafka and Python. Simulate virtual temperature sensors that continuously publish readings to a Kafka topic. A Kafka consumer application will subscribe to this topic, process the incoming data stream, and flag any temperature readings that exceed a defined threshold. This activity demonstrates Kafka's role in real-time IoT data ingestion, stream processing, and anomaly detection.

**Instructions:**

**A. Prerequisites**

 - Kafka and Zookeeper installed and running locally.

 - Python installed.

 - Kafka Python client installed:

```bash
pip install kafka-python

```
Create the Kafka topic:


```bash
kafka-topics.sh --create --topic temperature-readings \
  --bootstrap-server localhost:9092 \
  --partitions 1 --replication-factor 1
```

**B. Create the Kafka Producer (Python):**

- Simulate log messages (e.g., INFO, ERROR, DEBUG) using random or timestamped data.
- Send these messages continuously to the Kafka topic.

**C. Create the Kafka Consumer (Python):**

- Subscribe to the same Kafka topic.
- Continuously receive and print/log the incoming messages in real-time.

**D. Run the System**

---

### Activity 2. Building a Simple Real-Time Log Streaming System with Kafka and Python

**Objective:**
To implement a basic real-time data pipeline using Apache Kafka and Python by setting up a Kafka producer to continuously generate and send log messages to a Kafka topic, and a Kafka consumer to subscribe to the topic and display those messages in real time—laying the foundation for scalable, event-driven systems.

**Prerequisites**
Kafka installed locally (or use Confluent Cloud) with topic `orders-topic` and 3 partitions

Programming language SDK:

For Python: `confluent_kafka`

For Java: `kafka-clients`

**Instructions:**

A. Set up Kafka Producer

B. Set up Kafka Consumer

---

### Activity 3. Asynchronous Order Processing with Kafka in an E-Commerce Simulation

**Objective:**
To build a lightweight e-commerce simulation that demonstrates how Apache Kafka can decouple services by publishing user orders in real time and asynchronously processing them through independent inventory and notification services—improving responsiveness and scalability under high load.

**Instructions:**

**A. Set Up Kafka Locally or via Cloud Provider**

 - Install Apache Kafka and Zookeeper.

 - Create a topic named `orders-topic`.

**B. Create the Order Service (Producer)**

 - Simulate real-time user order events (e.g., user ID, product ID, quantity).

 - Publish each order message to the Kafka `orders` topic.

 - This should happen quickly without waiting for downstream processing.

**C. Implement the Inventory Service (Consumer 1)**

 - Consume messages from the `orders` topic.

 - Simulate inventory update (e.g., reduce stock count for ordered product).

 - Log each inventory adjustment.

**D. Implement the Notification Service (Consumer 2)**

 - Also consume from the `orders` topic.

 - Simulate sending a confirmation message to the user (e.g., via email/SMS log).

 - Ensure it works independently of the inventory service.

**E. Test the Workflow**

 - Run the producer and both consumers simultaneously.

 - Observe that the order is processed by multiple services in parallel, not sequentially.

-----

### References

- engineering.linkedin.com. (n.d.). *Kafka at LinkedIn: Current and Future*. [online] Available at: https://engineering.linkedin.com/kafka/kafka-linkedin-current-and-future

- Confluent. (n.d.). *How Netflix Uses Kafka for Distributed Streaming*. [online] Available at: https://www.confluent.io/blog/how-kafka-is-used-by-netflix/

- Ray (2025). *The Technology Behind Uber Real Time Ride Matching and Dynamic Pricing*. [online] Sde Ray - Tech Blogs. Available at: https://sderay.com/the-technology-behind-uber-real-time-ride-matching-and-dynamic-pricing/

- Waehner, K. (2022). *Fraud Detection with Apache Kafka, KSQL and Apache Flink - Kai Waehner*. [online] Kai Waehner. Available at: https://www.kai-waehner.de/blog/2022/10/25/fraud-detection-with-apache-kafka-ksql-and-apache-flink/

- Confluent. (2020). *Confluent, MQTT, and Apache Kafka Power Real-Time IoT Use Cases*. [online] Available at: https://www.confluent.io/blog/iot-streaming-use-cases-with-kafka-mqtt-confluent-and-waterstream/

-----
