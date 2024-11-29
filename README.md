# Fetch Data Engineering Take Home Solution

## Overview
This project implements a real-time streaming data pipeline using Kafka and Docker. The pipeline processes streaming data from a Kafka topic (`user-login`), applies transformations, and forwards the processed data to another Kafka topic (`processed-data`).

## Prerequisites
1. Docker installed on your system ([Download Docker](https://www.docker.com/)).
2. Python (version 3.7 or higher) installed.
3. Python dependencies installed: `kafka-python`.

## Project Setup

### Step 1: Clone the Repository
Clone the repository containing this project:
```bash
git clone <repository-link>
cd <repository-folder>
```

### Step 2: Set Up Docker Environment

#### 1. Create the `docker-compose.yml` file
Ensure you have the following content in a file named `docker-compose.yml`:
```yaml
version: '2'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - 22181:2181
    networks:
      - kafka-network

  kafka:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    ports:
      - 9092:9092
      - 29092:29092
    networks:
      - kafka-network
    environment:
      KAFKA_BROKER_ID: 0
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: LISTENER_INTERNAL://kafka:9092,LISTENER_EXTERNAL://localhost:29092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: LISTENER_INTERNAL:PLAINTEXT,LISTENER_EXTERNAL:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: LISTENER_INTERNAL
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1

  my-python-producer:
    image: mpradeep954/fetch-de-data-gen
    depends_on:
      - kafka
    restart: on-failure:10
    ports:
      - 9093:9093
    environment:
      BOOTSTRAP_SERVERS: kafka:9092
      KAFKA_TOPIC: user-login
    networks:
      - kafka-network

networks:
  kafka-network:
    driver: bridge
```

#### 2. Start the Docker environment
Run the following command to start the Kafka environment:
```bash
docker-compose up -d
```

#### 3. Verify Kafka Setup
Check if the `user-login` topic is receiving messages:
```bash
docker exec -it <kafka-container-id> kafka-console-consumer --bootstrap-server localhost:29092 --topic user-login --from-beginning
```

### Step 3: Run the Kafka Pipeline Script

#### 1. Install Python Dependencies
Ensure `kafka-python` is installed:
```bash
pip install kafka-python
```

#### 2. Save and Run the Script
Save the Python script from this project as `kafka_pipeline.py` and execute it:
```bash
python kafka_pipeline.py
```

#### 3. Verify Processed Data
Check if the processed data is being sent to the `processed-data` topic:
```bash
docker exec -it <kafka-container-id> kafka-console-consumer --bootstrap-server localhost:29092 --topic processed-data --from-beginning
```

## Design Choices

### Data Flow
1. **Kafka Consumer**: Reads messages from the `user-login` topic.
2. **Processing**: Applies transformations to the data (e.g., adding a `processed_time` field).
3. **Kafka Producer**: Publishes the processed data to the `processed-data` topic.

### Fault Tolerance
- **Error Handling**: Invalid or corrupt messages are logged, and processing continues.
- **Retries**: Built-in retries for Kafka producer.

### Scalability
- Multiple consumer instances can process data in parallel by using Kafka’s partitioning.
- The system can be deployed with a distributed architecture (e.g., Kubernetes).

## Production Deployment
1. **Orchestration**: Use Kubernetes to manage containers and ensure high availability.
2. **Monitoring**: Integrate with monitoring tools like Prometheus and Grafana.
3. **CI/CD**: Implement CI/CD pipelines for automated builds and deployments.

## Conclusion
This project demonstrates a functional real-time data pipeline using Kafka and Docker, showcasing efficient data ingestion, processing, and publishing. Follow the steps to replicate and run the solution.
