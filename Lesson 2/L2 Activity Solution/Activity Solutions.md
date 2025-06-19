### Activity 1. Real-Time Log Processing System with Kafka and Python

**A. Set Up Kafka Locally or Use a Cloud Provider:**

- Install and configure Apache Kafka and Zookeeper.
- Create a Kafka topic (e.g., `logs-topic`)
  
```bash
kafka-topics.sh --create --topic logs-topic --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
```

**B. Implement the Kafka Producer (Python):**

```bash
pip install kafka-python
```
`producer.py`

```python
from kafka import KafkaProducer
import time
import random

# Initialize producer
producer = KafkaProducer(bootstrap_servers='localhost:9092')

# Log levels for simulation
log_levels = ['INFO', 'ERROR', 'DEBUG', 'WARNING']

# Produce log messages
while True:
    log_level = random.choice(log_levels)
    timestamp = time.strftime("%Y-%m-%d %H:%M:%S")
    message = f"{log_level}: Log message at {timestamp}"
    
    producer.send('logs-topic', message.encode('utf-8'))
    print(f"Sent: {message}")
    
    time.sleep(2)
```

**C. Implement the Kafka Consumer (Python):**

`consumer.py`

```python
from kafka import KafkaConsumer

# Initialize consumer
consumer = KafkaConsumer(
    'logs-topic',
    bootstrap_servers='localhost:9092',
    auto_offset_reset='earliest',
    group_id='log-consumers'
)

# Consume and print log messages
for msg in consumer:
    print(f"Received: {msg.value.decode('utf-8')}")
```

**D. Ensure Fault Tolerance and Robustness:**

 - Use `try-except` blocks in both producer and consumer to handle exceptions.

 - Consider adding retries, logging to a file, or buffering messages in case of failure.

**E. How to Run:**

 - Start Kafka and Zookeeper.

 - Create the topic `logs-topic`.

In one terminal, run:

```bash
python consumer.py
```

In another terminal, run:

```bash
python producer.py
```

Watch logs stream in real time!

---

### Activity 2. Building a Simple Real-Time Log Streaming System with Kafka and Python

**Objective:**
To implement a basic real-time data pipeline using Apache Kafka and Python by setting up a Kafka producer to continuously generate and send log messages to a Kafka topic, and a Kafka consumer to subscribe to the topic and display those messages in real time—laying the foundation for scalable, event-driven systems.

**Instructions:**

**A. Set up Kafka Producer:** 
```python
from kafka import KafkaProducer
import time

# Initialise the Kafka producer
producer = KafkaProducer(bootstrap_servers='localhost:9092')

# Continuously send log messages
while True:
    log_msg = "Log message at " + time.strftime("%Y-%m-%d %H:%M:%S")
    producer.send('logs', log_msg.encode('utf-8'))
    print("Sent:", log_msg)
    time.sleep(2)

```

**B. Set up Kafka Consumer:**

```python
from kafka import KafkaConsumer

# Initialize the Kafka consumer
consumer = KafkaConsumer('logs', bootstrap_servers='localhost:9092')

# Continuously listen for messages and print them
for message in consumer:
    print("Received:", message.value.decode('utf-8'))
```

---

### Activity 3. Asynchronous Order Processing with Kafka in an E-Commerce Simulation**

**1. Prerequisites**
Kafka installed locally (or use Confluent Cloud) with topic `orders-topic` and 3 partitions

Programming language SDK:

For Python: `confluent_kafka`

For Java: `kafka-clients` 

**2. Producer Code Example (Python)**

```python
from confluent_kafka import Producer
import json
import uuid
import time

# Set up the Kafka producer
p = Producer({'bootstrap.servers': 'localhost:9092'})

# Callback function for delivery reports
def delivery_report(err, msg):
    if err:
        print(f"Delivery failed: {err}")
    else:
        print(f"Delivered to {msg.topic()} [{msg.partition()}]")

# Produce 10 sample orders
for i in range(10):
    order = {
        'order_id': str(uuid.uuid4()),
        'user_id': f'user_{i%3}',
        'items': ['itemA', 'itemB'],
        'total': round(100 + i * 10, 2)
    }
    p.produce('orders-topic', key=order['user_id'], value=json.dumps(order), callback=delivery_report)
    p.poll(0)
    time.sleep(1)

# Wait for all messages to be delivered
p.flush()
`````
 **3. Consumer Code Example (Python for Inventory)**

 ```python
from confluent_kafka import Consumer
import json

# Set up the Kafka consumer
c = Consumer({
    'bootstrap.servers': 'localhost:9092',
    'group.id': 'inventory-service',
    'auto.offset.reset': 'earliest'
})

# Subscribe to the topic
c.subscribe(['orders-topic'])

# Consume messages
while True:
    msg = c.poll(1.0)
    if msg is None:
        continue
    if msg.error():
        print(msg.error())
        continue

    order = json.loads(msg.value())
    print(f"Processing inventory for order {order['order_id']} (User {order['user_id']})")
    # Simulate inventory update
    c.commit()
 `````

**4. Notification Service Consumer**

Use the same structure as the Inventory Service Consumer, but change the group ID:

```python
'group.id': 'notification-service'
`````
Then update the print message to simulate sending a confirmation:

```python
print(f"Sending confirmation for order {order['order_id']} to User {order['user_id']}")
`````

**5. Running the Project**

 - Start Kafka (and ZooKeeper if needed).

 - Create the Kafka topic with 3 partitions:

```bash
kafka-topics --create --topic orders-topic --bootstrap-server localhost:9092 --partitions 3 --replication-factor 1
`````
 - Launch Inventory and Notification consumers (in separate terminals or threads).

 - Run the Producer script to simulate real-time orders.

 - Observe that both consumers independently process the same order messages from the topic.
