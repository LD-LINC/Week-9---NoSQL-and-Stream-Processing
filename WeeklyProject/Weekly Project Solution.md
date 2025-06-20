# Solution

**1. Install & Start Kafka and MongoDB. Start both services locally**

**Start Zookeeper and Kafka**

```bash
bin/zookeeper-server-start.sh config/zookeeper.properties
bin/kafka-server-start.sh config/server.properties
```
**Create Kafka topic**
```bash
bin/kafka-topics.sh --create --topic nyc311 --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
```
---
**2. Prepare Dataset for Streaming.**
 - Save the downloaded file as `311_service_requests.csv`.
 - You’ll read this dataset in chunks (e.g., 100 rows at a time), mimicking a real-time feed.
---
**3. Kafka Producer - Simulate Real-Time Streaming**

```python
# producer.py
import pandas as pd
import json
import time
from kafka import KafkaProducer

producer = KafkaProducer(
    bootstrap_servers='localhost:9092',
    value_serializer=lambda x: json.dumps(x).encode('utf-8')
)

df = pd.read_csv('311_service_requests.csv', usecols=[
    'Unique Key', 'Created Date', 'Complaint Type', 'Descriptor',
    'Borough', 'Latitude', 'Longitude'
])

# Simulate real-time stream in chunks
chunk_size = 100

for i in range(0, len(df), chunk_size):
    chunk = df.iloc[i:i+chunk_size]
    for _, row in chunk.iterrows():
        message = row.dropna().to_dict()
        producer.send('nyc311', message)
        print(f"Produced: {message.get('Complaint Type')}")
    time.sleep(2)  # Wait between chunks
```
---
**4. Kafka Consumer - Store to MongoDB**

```python
# consumer.py
from kafka import KafkaConsumer
from pymongo import MongoClient
import json

consumer = KafkaConsumer(
    'nyc311',
    bootstrap_servers='localhost:9092',
    auto_offset_reset='earliest',
    enable_auto_commit=True,
    value_deserializer=lambda m: json.loads(m.decode('utf-8'))
)

mongo_client = MongoClient("mongodb://localhost:27017/")
db = mongo_client["nyc311"]
collection = db["complaints"]

for msg in consumer:
    complaint = msg.value
    collection.insert_one(complaint)
    print(f"Inserted: {complaint.get('Complaint Type')} into MongoDB")
```
---
**5. Verify MongoDB Insertions**

```bash
# In Mongo shell
use nyc311
db.complaints.countDocuments()
db.complaints.find().limit(2).pretty()
```

