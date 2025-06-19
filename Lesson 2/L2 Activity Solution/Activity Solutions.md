### Activity 1. Real-Time Log Processing System with Kafka and Python

**Objective:**
Build a real-time temperature monitoring system using Apache Kafka and Python. Simulate virtual temperature sensors that continuously publish readings to a Kafka topic. A Kafka consumer application will subscribe to this topic, process the incoming data stream, and flag any temperature readings that exceed a defined threshold. This activity demonstrates Kafka's role in real-time IoT data ingestion, stream processing, and anomaly detection.

**Solution:**

**1: Simulate Temperature Sensor (Producer)**

```python
from kafka import KafkaProducer
import random
import time

producer = KafkaProducer(bootstrap_servers='localhost:9092')
while True:
    temp = round(random.uniform(20.0, 40.0), 2)
    producer.send('temperature', str(temp).encode('utf-8'))
    print("Sent Temperature:", temp)
    time.sleep(3)
```

**2: Consumer to Detect High Temperature**

```python
from kafka import KafkaConsumer

consumer = KafkaConsumer('temperature', bootstrap_servers='localhost:9092')
for message in consumer:
    temp = float(message.value.decode('utf-8'))
    if temp > 35:
        print(f"ALERT: High temperature detected - {temp}°C")
    else:
        print("Temperature OK:", temp)
```

---

### Activity 2. Building a Simple Real-Time Log Streaming System with Kafka and Python

**Objective:**
To implement a basic real-time data pipeline using Apache Kafka and Python by setting up a Kafka producer to continuously generate and send log messages to a Kafka topic, and a Kafka consumer to subscribe to the topic and display those messages in real time—laying the foundation for scalable, event-driven systems.

**Solution:**

**1. Set up Kafka Producer:** 
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

**2. Set up Kafka Consumer:**

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

**Solution:** 

**1. Producer Code Example (Python)**

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
 **2. Consumer Code Example (Python for Inventory)**

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

**3. Notification Service Consumer**

Use the same structure as the Inventory Service Consumer, but change the group ID:

```python
'group.id': 'notification-service'
`````
Then update the print message to simulate sending a confirmation:

```python
print(f"Sending confirmation for order {order['order_id']} to User {order['user_id']}")
`````

**4. Running the Project**

 - Start Kafka (and ZooKeeper if needed).

 - Create the Kafka topic with 3 partitions:

```bash
kafka-topics --create --topic orders-topic --bootstrap-server localhost:9092 --partitions 3 --replication-factor 1
`````
 - Launch Inventory and Notification consumers (in separate terminals or threads).

 - Run the Producer script to simulate real-time orders.

 - Observe that both consumers independently process the same order messages from the topic.
