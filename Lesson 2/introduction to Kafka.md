## Lesson 2:  Introduction to Kafka

### What is Kafka?

Apache Kafka is an open-source distributed streaming platform that enables you to publish, subscribe to, store, and process streams of records in a fault-tolerant way. It's often described as a distributed commit log or a distributed publish-subscribe messaging system. Unlike traditional messaging systems, Kafka is designed for high-throughput, low-latency data ingestion, and processing, making it ideal for real-time data pipelines and streaming applications.

### Why Kafka? (Use Cases and Benefits)

Kafka's popularity stems from its ability to address common challenges in data integration and processing. Here are some key benefits and typical use cases:

**Benefits:**

<p align="center">
  <img src="Lesson 2/L2_Assests/Benefits of Kafka - visual selection.png" alt="Benefits of Kafka" width="80%" />
</p>

**Use Cases:**

  * **Real-time Analytics:** Ingesting and processing sensor data, website activity, or financial transactions for immediate insights.
  * **Log Aggregation:** Centralizing logs from various applications and servers for monitoring and analysis.
  * **Event Sourcing:** Storing all changes to an application's state as a sequence of immutable events.
  * **Stream Processing:** Building applications that continuously process streams of data (e.g., fraud detection, anomaly detection).
  * **Message Queues:** A high-throughput alternative to traditional message queues for decoupling applications.
  * **Data Integration:** Connecting various systems by acting as a central hub for data exchange.

### Brief History of Kafka

Kafka was originally developed at LinkedIn in 2011 to handle the company's massive volume of activity stream data. It was designed to provide a high-throughput, low-latency platform for handling event streams at scale. In 2012, it was open-sourced under the Apache Software Foundation, and since then, it has grown into a vibrant open-source community with widespread adoption across various industries.

-----

## 2. Kafka Core Concepts & Architecture

Understanding Kafka's core concepts is crucial for effective utilization.

### Distributed Commit Log

At its heart, Kafka can be thought of as a **distributed commit log**. This means it appends records (messages) to a log in an ordered, immutable fashion. This log is distributed across multiple servers (brokers) for scalability and fault tolerance. Each record in the log is assigned a unique, sequential ID called an **offset**.

### Topics, Partitions, and Offsets

  * **Topic:** A named logical channel to which records are published. Think of a topic as a category or feed name where messages of a particular type are sent. For example, you might have a topic for "user\_signups" or "payment\_transactions."

  * **Partition:** A topic is divided into one or more partitions. Each partition is an ordered, immutable sequence of records. Records within a partition are strictly ordered, but there is no guaranteed order across different partitions of the same topic. Partitions are the unit of parallelism in Kafka. More partitions allow for higher throughput.

  * **Offset:** Each record within a partition is assigned a unique, sequential ID called an offset. This offset acts as a pointer to a specific record within a partition. Consumers use offsets to track their position within a partition, ensuring they consume messages in the correct order and don't miss any.

    ```
    Topic A
    ├── Partition 0 [Record 0, Record 1, Record 2, ...]
    └── Partition 1 [Record 0, Record 1, Record 2, ...]
    ```
<p align="center">
  <img src="L2_Assests/Kafka System.PNG" alt="Core Concept" width="60%" />
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

### ZooKeeper (and its role in Kafka)

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

## 3. Kafka Components

Beyond the core concepts, Kafka offers several powerful components that extend its functionality.

### Kafka Connect

**Kafka Connect** is a framework for building and running scalable, reliable data pipelines. It simplifies the process of integrating Kafka with other systems (databases, file systems, message queues, etc.). It provides two types of connectors:

  * **Source Connectors:** Ingest data from external systems into Kafka topics.
  * **Sink Connectors:** Deliver data from Kafka topics to external systems.

Kafka Connect handles much of the boilerplate code for data integration, including fault tolerance, scalability, and offset management.

### Kafka Streams

**Kafka Streams** is a client-side library for building real-time stream processing applications. It allows you to transform, filter, aggregate, and join data streams directly within your application code. Key features include:

  * **Low Latency:** Processes data as it arrives with minimal delay.
  * **Fault Tolerance:** Built-in fault tolerance and state management.
  * **Scalability:** Scales horizontally by leveraging Kafka's consumer groups.
  * **Exactly-once Processing:** Guarantees that each record is processed exactly once, even in the event of failures.

Kafka Streams makes it easy to build powerful streaming applications without needing a separate distributed processing framework.

### Kafka Clients (APIs)

Kafka provides official client libraries (APIs) for various programming languages to interact with the cluster:

  * **Java Client:** The primary and most mature client.
  * **Python Client:** (e.g., `kafka-python`, `confluent-kafka-python`)
  * **Go Client:** (e.g., `confluent-kafka-go`)
  * **Node.js Client:** (e.g., `kafkajs`)
  * ... and many more community-contributed clients.

These clients abstract away the complexities of interacting with the Kafka protocol, allowing developers to focus on their application logic.

### Schema Registry (Confluent Platform)

While not strictly part of Apache Kafka, the **Schema Registry** (part of Confluent Platform, a company founded by the creators of Kafka) is a highly recommended component for production environments. It provides a centralized repository for managing schemas (data structures) for your Kafka messages.

  * **Schema Evolution:** Enables backward and forward compatibility for schemas, allowing applications to evolve without breaking existing consumers.
  * **Data Governance:** Ensures data consistency and quality by enforcing schema adherence.
  * **Serialization/Deserialization:** Works seamlessly with serializers like Avro, Protobuf, and JSON Schema to automatically handle data conversion.

-----

## 4. Kafka Setup & CLI

Let's get our hands dirty with a basic Kafka setup and some command-line interface (CLI) operations. This will typically be a local setup for development and learning.

### Prerequisites (Java)

Kafka is written in Scala and runs on the Java Virtual Machine (JVM). Therefore, you need to have Java installed on your system.

  * **Java Development Kit (JDK) 8 or higher** is generally recommended.

You can verify your Java installation by running:

```bash
java -version
```

### Downloading and Extracting Kafka

1.  **Download:** Go to the official Apache Kafka website: [https://kafka.apache.org/downloads](https://kafka.apache.org/downloads)
    Choose the latest stable release (e.g., `kafka_2.13-3.x.x.tgz`). The `2.13` refers to the Scala version.

2.  **Extract:**

    ```bash
    tar -xzf kafka_2.13-3.x.x.tgz
    cd kafka_2.13-3.x.x
    ```

    Let's assume you're in the extracted Kafka directory for the following commands.

### Starting ZooKeeper

Kafka relies on ZooKeeper (unless you are using Kafka's Raft-based KRaft mode, which is still evolving for production). For a quick start, we'll use the bundled ZooKeeper.

```bash
bin/zookeeper-server-start.sh config/zookeeper.properties
```

You should see ZooKeeper starting up and logging messages to the console. Keep this terminal window open.

### Starting Kafka Broker

In a new terminal window, start the Kafka broker.

```bash
bin/kafka-server-start.sh config/server.properties
```

You'll see Kafka starting up and logging messages. Once it says something like `[KafkaServer id=0] started`, your Kafka broker is running.

### Basic CLI Commands

Now that Kafka is running, let's interact with it using the command-line tools.

#### Creating Topics

Let's create a topic named `my_first_topic` with 3 partitions and a replication factor of 1 (for a single-broker setup, replication factor > 1 won't work without more brokers).

```bash
bin/kafka-topics.sh --create --topic my_first_topic --bootstrap-server localhost:9092 --partitions 3 --replication-factor 1
```

You should see a message indicating the topic was created.

#### Listing Topics

To see all topics in your Kafka cluster:

```bash
bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```

You should see `my_first_topic` in the list.

#### Producing Messages

Let's send some messages to `my_first_topic` using the console producer.

```bash
bin/kafka-console-producer.sh --topic my_first_topic --bootstrap-server localhost:9092
```

After running this command, you'll get a `>` prompt. Type your messages and press Enter after each one:

```
>Hello Kafka!
>This is my first message.
>Learning Kafka is fun!
```

Press `Ctrl+C` to exit the producer.

#### Consuming Messages

Now, let's consume the messages we just produced using the console consumer. The `--from-beginning` flag tells the consumer to start reading from the very beginning of the topic's log.

```bash
bin/kafka-console-consumer.sh --topic my_first_topic --bootstrap-server localhost:9092 --from-beginning
```

You should see the messages you typed earlier printed to the console:

```
Hello Kafka!
This is my first message.
Learning Kafka is fun!
```

Keep this consumer running. If you send new messages from the producer, they will appear here in real-time. Press `Ctrl+C` to exit the consumer.

#### Deleting Topics

You can delete a topic, but often you need to enable topic deletion in `config/server.properties` by setting `delete.topic.enable=true` and restarting the broker.

```bash
bin/kafka-topics.sh --delete --topic my_first_topic --bootstrap-server localhost:9092
```

You might get a message saying it's marked for deletion.

-----

## 5. Kafka with Python

Python is a popular language for interacting with Kafka due to its simplicity and extensive library ecosystem. We'll use the `kafka-python` library for these examples.

### Setting up the Python Environment

It's good practice to use a virtual environment for your Python projects.

```bash
python3 -m venv kafka_env
source kafka_env/bin/activate # On Windows: .\kafka_env\Scripts\activate
```

### Installing `kafka-python`

```bash
pip install kafka-python
```

### Basic Producer Example

Create a file named `producer.py`:
```python
from kafka import KafkaProducer
import json
import time

# Create a KafkaProducer instance
# bootstrap_servers specifies the Kafka broker addresses
producer = KafkaProducer(
    bootstrap_servers='localhost:9092',
    value_serializer=lambda v: json.dumps(v).encode('utf-8') # Serialize messages to JSON bytes
)

topic_name = 'python_topic'

print(f"Producing messages to topic: {topic_name}")

for i in range(5):
    message = {'number': i, 'timestamp': time.time()}
    print(f"Sending: {message}")
    producer.send(topic_name, message)
    time.sleep(1)

# Ensure all messages are sent before closing the producer
producer.flush()
print("Messages sent successfully!")
producer.close()
```
Run the producer:

```bash
python producer.py
```

### Basic Consumer Example

Create a file named `consumer.py`:
```python
from kafka import KafkaConsumer
import json

topic_name = 'python_topic'

# Create a KafkaConsumer instance
consumer = KafkaConsumer(
    topic_name,
    bootstrap_servers='localhost:9092',
    auto_offset_reset='earliest', # Start reading from the beginning if no offset is found
    enable_auto_commit=True,      # Automatically commit offsets
    group_id='my_python_group',   # Consumer group ID
    value_deserializer=lambda x: json.loads(x.decode('utf-8')) # Deserialize messages from JSON bytes
)

print(f"Consuming messages from topic: {topic_name}")

try:
    for message in consumer:
        # message is a ConsumerRecord object
        print(f"Received message: Partition={message.partition}, Offset={message.offset}, Value={message.value}")
except KeyboardInterrupt:
    print("Consumer stopped by user.")
finally:
    consumer.close()
```
Before running the consumer, make sure you've created `python_topic` using the CLI:

```bash
bin/kafka-topics.sh --create --topic python_topic --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
```

Then run the consumer:

```bash
python consumer.py
```

You should see the messages produced by `producer.py`. If `producer.py` is still running or you run it again, the consumer will receive new messages in real-time.

### Working with Consumer Groups in Python

The `consumer.py` example already demonstrates a basic consumer group usage with `group_id='my_python_group'`. If you run multiple instances of `consumer.py` with the *same* `group_id`, they will form a consumer group and collectively share the partitions of `python_topic`. If you run them with different `group_id`s, they will each consume all messages independently.

### Handling Serialization/Deserialization

In the examples above, we used `json.dumps()` and `json.loads()` for serialization and deserialization. Kafka messages are essentially byte arrays. You need to decide on a serialization format (e.g., JSON, Avro, Protobuf, simple strings) that your producers and consumers agree upon.

  * **`value_serializer`** in `KafkaProducer`: A function that takes your message object and returns bytes.
  * **`value_deserializer`** in `KafkaConsumer`: A function that takes bytes and returns your message object.

For more robust schema management, especially in production, consider integrating with a Schema Registry and using libraries that support it (like `confluent-kafka-python` with Avro serializers).

-----

## 6. Security, Monitoring & Tuning

Once you move beyond local development, these aspects become critical for a production-ready Kafka cluster.

### Security

Securing your Kafka cluster is paramount to protect your data.

  * **Authentication (SASL):** Verifies the identity of clients (producers and consumers) and brokers. Common mechanisms include:
      * **SASL/PLAIN:** Username/password authentication (simple but plaintext credentials).
      * **SASL/SCRAM:** Secure password-based authentication (recommended over PLAIN).
      * **SASL/GSSAPI (Kerberos):** Enterprise-grade authentication for large organizations.
      * **SASL/OAUTHBEARER:** Token-based authentication.
  * **Authorization (ACLs):** Controls what authenticated clients are allowed to do (e.g., which topics they can read from or write to). Kafka provides Access Control Lists (ACLs) to define permissions.
  * **Encryption (SSL/TLS):** Encrypts data in transit between clients and brokers, and between brokers themselves, preventing eavesdropping and tampering.

Implementing security involves configuring `server.properties` on brokers, and `producer.properties`/`consumer.properties` for clients.

### Monitoring

Effective monitoring is crucial to ensure the health, performance, and stability of your Kafka cluster.

  * **JMX Metrics:** Kafka exposes a rich set of JMX (Java Management Extensions) metrics. These provide detailed information about broker performance, topic throughput, consumer lag, and more.
  * **Kafka Tooling:**
      * **Kafka Manager (Yahoo!):** A popular open-source tool for managing and monitoring Kafka clusters. Provides a web UI for viewing topics, consumers, and broker metrics.
      * **Confluent Control Center:** A comprehensive web-based management system for Confluent Platform (includes Kafka), offering advanced monitoring, schema management, and stream governance.
  * **External Monitoring Solutions:**
      * **Prometheus & Grafana:** A widely used combination for collecting and visualizing time-series data. Prometheus scrapes JMX exporter metrics from Kafka, and Grafana creates dashboards.
      * **Datadog, New Relic, Splunk:** Commercial monitoring solutions that offer Kafka integrations.

Key metrics to monitor:

  * **Broker health:** CPU, memory, disk usage, network I/O.
  * **Topic throughput:** Bytes in/out, message rate.
  * **Consumer lag:** The number of messages a consumer group is behind the latest message in a partition. This is a critical indicator of consumer health.
  * **Replication status:** Under-replicated partitions.

### Tuning

Optimizing Kafka performance involves tuning various configurations on brokers, producers, and consumers, as well as the underlying operating system.

  * **Broker Configurations (`server.properties`):**
      * `num.partitions`: Number of partitions per topic (affects parallelism).
      * `log.retention.hours`/`log.retention.bytes`: How long/much data to retain.
      * `default.replication.factor`: Default replication factor for new topics.
      * `num.network.threads`, `num.io.threads`: Number of threads for network and I/O operations.
      * `log.segment.bytes`, `log.segment.ms`: Size and time thresholds for log segments.
  * **Producer Configurations:**
      * `acks`: Durability level for messages (0, 1, or "all"). "all" ensures message is committed to all in-sync replicas.
      * `batch.size`: Number of bytes producer will attempt to batch together before sending.
      * `linger.ms`: Time producer will wait before sending records, even if `batch.size` is not met.
      * `compression.type`: Compression algorithm (gzip, snappy, lz4, zstd) for better throughput.
  * **Consumer Configurations:**
      * `fetch.min.bytes`: Minimum data a consumer will wait for from a broker.
      * `fetch.max.wait.ms`: Maximum time a consumer will wait for `fetch.min.bytes`.
      * `session.timeout.ms`: How long a consumer can be out of contact with the broker before being considered dead.
      * `max.poll.records`: Maximum number of records returned in a single `poll()` call.
      * `auto.offset.reset`: What to do when no initial offset is found or if the current offset is invalid (`earliest`, `latest`, `none`).
  * **Operating System Tuning:**
      * **File Descriptors:** Increase the maximum number of open file descriptors.
      * **Disk I/O:** Use fast disks (SSDs) and consider RAID configurations.
      * **Network:** Optimize network buffer sizes and ensure sufficient bandwidth.
      * **Memory:** Allocate sufficient memory to the JVM.

Tuning is an iterative process that requires understanding your workload and monitoring the impact of changes.

-----

## 7. Case Studies

Kafka's versatility has led to its adoption by numerous companies across various industries for critical use cases.

### LinkedIn (Originators of Kafka)

  * **Origin:** Kafka was born at LinkedIn to handle the massive volume of user activity data (page views, likes, shares, etc.).
  * **Use Cases:**
      * **Activity Stream Data:** Centralizing and processing real-time activity streams for personalized recommendations, news feeds, and analytics.
      * **Metrics & Monitoring:** Ingesting operational metrics from thousands of servers for real-time dashboards and alerting.
      * **Data Integration:** Connecting various internal systems and databases.

### Netflix

  * **Use Cases:**
      * **Real-time Monitoring & Alerting:** Collecting and analyzing telemetry data from their vast infrastructure for operational insights and anomaly detection.
      * **Playback Tracking:** Tracking user playback events for personalization, billing, and content recommendations.
      * **Log Aggregation:** Centralizing logs from all services for debugging and analysis.
      * **Data Pipelining:** Moving data between various microservices and data stores.

### Uber

  * **Use Cases:**
      * **Real-time Ride Matching:** Processing location updates from drivers and riders to match them efficiently.
      * **Fraud Detection:** Analyzing transaction streams in real-time to detect and prevent fraudulent activities.
      * **Surge Pricing:** Dynamic pricing adjustments based on real-time demand and supply data.
      * **Operational Intelligence:** Collecting and processing data for real-time analytics on driver availability, trip status, and more.

### Financial Institutions (Example Use Cases)

Many financial institutions leverage Kafka for its high throughput and durability, especially for sensitive data.

  * **Real-time Transaction Processing:** Ingesting and processing millions of financial transactions per second for fraud detection, risk management, and regulatory compliance.
  * **Market Data Feeds:** Distributing real-time stock prices, exchange rates, and other market data to trading applications.
  * **Customer 360 View:** Aggregating customer interactions from various channels (web, mobile, call center) to create a unified customer profile for personalized services.
  * **Auditing and Compliance:** Maintaining immutable logs of all events for regulatory auditing and forensic analysis.

### Real-time Analytics Pipelines

This is a broad category, but Kafka is a foundational component in many modern real-time analytics architectures.

  * **ETL (Extract, Transform, Load) Pipelines:** Acting as a central nervous system for data movement, ingesting data from various sources, transforming it using Kafka Streams or other stream processing frameworks, and loading it into data warehouses or data lakes for analysis.
  * **Recommendation Engines:** Feeding user behavior data into real-time recommendation systems.
  * **IoT Data Ingestion:** Collecting massive volumes of data from IoT devices for monitoring, predictive maintenance, and operational optimization.

These case studies highlight Kafka's flexibility and power as a central nervous system for data, enabling organizations to build highly scalable, resilient, and real-time data-driven applications.

-----
