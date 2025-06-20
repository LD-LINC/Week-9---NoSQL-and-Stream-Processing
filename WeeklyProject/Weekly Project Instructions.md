
### Real-Time Urban Complaint Monitoring with Kafka, Python, and MongoDB
In modern urban environments, timely and efficient handling of public complaints is essential for city administration. The NYC 311 service provides a platform for citizens to report non-emergency issues ranging from noise complaints to infrastructure defects. However, the traditional methods of storing and analyzing such massive volumes of data often fall short in terms of scalability and responsiveness.

**Objective:** To build a real-time complaint management simulation using Apache Kafka, Python, and MongoDB to process NYC 311 service request data. By simulating real-time ingestion and storage of over 3 million complaint records, this project demonstrates how modern big data tools can be integrated to address urban challenges with scalability, speed, and efficiency.

**Instructions:** 

**1. Download and Prepare the Dataset**
 - Download the dataset from Kaggle: [NYC 311 Service Requests Dataset](https://www.kaggle.com/datasets/new-york-city/ny-311-service-requests)

**2. Preprocess the data:**

 - Extract relevant fields: `Created Date`, `Complaint Type`, `Descriptor`, `Borough`, and `Location`.

 - Convert the CSV into manageable chunks or JSON format for simulated streaming.

**3. Set Up Apache Kafka**
 - Install and run Kafka and Zookeeper locally (or use a managed service like Confluent Cloud).
 - Create a Kafka topic named `nyc-311-requests`.

**4. Implement Kafka Producer in Python**
 - Use the `kafka-python or `confluent-kafka` library to send complaint data row by row to the `nyc-311-requests` topic.
 - Introduce a delay (time.sleep()`) to simulate real-time streaming.

**5. Implement Kafka Consumer in Python**
 - Create a consumer that listens to the `nyc-311-requests` topic and processes incoming records.
 - Insert each record into MongoDB.

**6. Verify Data Storage and Explore Queries**
 - Open MongoDB Compass or use the shell to verify that data is being inserted.
