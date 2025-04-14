# Running Apache Kafka with Docker: A Step-by-Step Guide

*Published on August 25, 2023*

## Introduction

Apache Kafka is a distributed event streaming platform used for high-performance data pipelines, streaming analytics, and data integration. Running Kafka in Docker containers offers numerous advantages, including simplified deployment, consistent environments, and easy version management.

This guide provides detailed, step-by-step instructions for setting up and running Apache Kafka using Docker, from basic setup to advanced configurations for production environments.

## Prerequisites

Before you begin, ensure you have the following installed:

- Docker Engine (version 19.03 or later)
- Docker Compose (version 1.27 or later) - required for multi-container setup
- At least 4GB of RAM allocated to Docker

You can verify your Docker installation by running:

```bash
docker --version
docker-compose --version
```

## Step 1: Understanding Kafka Architecture

Before diving into the Docker setup, it's important to understand the basic components of Kafka:

- **Broker**: The Kafka server that stores and serves messages
- **ZooKeeper**: Coordinates the Kafka cluster (Note: Kafka is moving towards removing this dependency)
- **Topic**: A category or feed name to which records are published
- **Partition**: Topics are divided into partitions for scalability
- **Producer**: Application that sends messages to Kafka
- **Consumer**: Application that reads messages from Kafka

## Step 2: Set Up a Single-Node Kafka with Docker Compose

The easiest way to get started with Kafka in Docker is to use Docker Compose. Create a file named `docker-compose.yml` with the following content:

```yaml
version: '3'

services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.3.0
    container_name: zookeeper
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - "2181:2181"

  kafka:
    image: confluentinc/cp-kafka:7.3.0
    container_name: kafka
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
```

This configuration sets up:
- A ZooKeeper container for Kafka coordination
- A single Kafka broker accessible on port 9092

## Step 3: Start the Kafka Environment

Start the Kafka environment using Docker Compose:

```bash
docker-compose up -d
```

This command starts both ZooKeeper and Kafka containers in detached mode.

Verify that the containers are running:

```bash
docker-compose ps
```

You should see both containers in the "Up" state.

## Step 4: Create a Kafka Topic

Create a topic named "test-topic" with 1 partition and a replication factor of 1:

```bash
docker exec -it kafka kafka-topics --create --topic test-topic --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
```

List all topics to verify that your topic was created:

```bash
docker exec -it kafka kafka-topics --list --bootstrap-server localhost:9092
```

## Step 5: Produce and Consume Messages

### Produce Messages

Start a Kafka console producer to send messages to the "test-topic":

```bash
docker exec -it kafka kafka-console-producer --topic test-topic --bootstrap-server localhost:9092
```

Type some messages, pressing Enter after each message. For example:

```
Hello, Kafka!
This is a test message.
Docker makes Kafka setup easy.
```

Press Ctrl+C to exit the producer.

### Consume Messages

Start a Kafka console consumer to read messages from the "test-topic":

```bash
docker exec -it kafka kafka-console-consumer --topic test-topic --from-beginning --bootstrap-server localhost:9092
```

You should see the messages you produced earlier. Press Ctrl+C to exit the consumer.

## Step 6: Set Up a Multi-Broker Kafka Cluster

For a more realistic setup, you might want to run a multi-broker Kafka cluster. Create a new `docker-compose-multi-broker.yml` file:

```yaml
version: '3'

services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.3.0
    container_name: zookeeper
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - "2181:2181"

  kafka1:
    image: confluentinc/cp-kafka:7.3.0
    container_name: kafka1
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka1:9092,PLAINTEXT_HOST://localhost:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 2
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3

  kafka2:
    image: confluentinc/cp-kafka:7.3.0
    container_name: kafka2
    depends_on:
      - zookeeper
    ports:
      - "9093:9093"
    environment:
      KAFKA_BROKER_ID: 2
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka2:9092,PLAINTEXT_HOST://localhost:9093
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 2
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3

  kafka3:
    image: confluentinc/cp-kafka:7.3.0
    container_name: kafka3
    depends_on:
      - zookeeper
    ports:
      - "9094:9094"
    environment:
      KAFKA_BROKER_ID: 3
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka3:9092,PLAINTEXT_HOST://localhost:9094
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 2
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3
```

Start the multi-broker Kafka cluster:

```bash
docker-compose -f docker-compose-multi-broker.yml up -d
```

## Step 7: Create a Replicated Topic

Create a topic with replication across the brokers:

```bash
docker exec -it kafka1 kafka-topics --create --topic replicated-topic --bootstrap-server kafka1:9092 --partitions 3 --replication-factor 3
```

Verify the topic configuration:

```bash
docker exec -it kafka1 kafka-topics --describe --topic replicated-topic --bootstrap-server kafka1:9092
```

You should see information about the topic, including the partition leaders and replicas.

## Step 8: Test Fault Tolerance

To test fault tolerance, you can stop one of the Kafka brokers and verify that the system still works:

```bash
docker stop kafka2
```

Produce and consume messages as before, but using the remaining brokers:

```bash
# Produce messages
docker exec -it kafka1 kafka-console-producer --topic replicated-topic --bootstrap-server kafka1:9092

# Consume messages
docker exec -it kafka3 kafka-console-consumer --topic replicated-topic --from-beginning --bootstrap-server kafka3:9092
```

Restart the stopped broker:

```bash
docker start kafka2
```

## Step 9: Set Up Kafka Connect

Kafka Connect is a tool for streaming data between Apache Kafka and other systems. Add Kafka Connect to your Docker Compose file:

```yaml
version: '3'

services:
  # ... ZooKeeper and Kafka services from previous example ...

  kafka-connect:
    image: confluentinc/cp-kafka-connect:7.3.0
    container_name: kafka-connect
    depends_on:
      - kafka1
      - kafka2
      - kafka3
    ports:
      - "8083:8083"
    environment:
      CONNECT_BOOTSTRAP_SERVERS: kafka1:9092,kafka2:9092,kafka3:9092
      CONNECT_REST_PORT: 8083
      CONNECT_GROUP_ID: kafka-connect
      CONNECT_CONFIG_STORAGE_TOPIC: _connect-configs
      CONNECT_OFFSET_STORAGE_TOPIC: _connect-offsets
      CONNECT_STATUS_STORAGE_TOPIC: _connect-status
      CONNECT_KEY_CONVERTER: org.apache.kafka.connect.json.JsonConverter
      CONNECT_VALUE_CONVERTER: org.apache.kafka.connect.json.JsonConverter
      CONNECT_INTERNAL_KEY_CONVERTER: org.apache.kafka.connect.json.JsonConverter
      CONNECT_INTERNAL_VALUE_CONVERTER: org.apache.kafka.connect.json.JsonConverter
      CONNECT_REST_ADVERTISED_HOST_NAME: kafka-connect
      CONNECT_LOG4J_ROOT_LOGLEVEL: INFO
      CONNECT_LOG4J_LOGGERS: org.reflections=ERROR
      CONNECT_CONFIG_STORAGE_REPLICATION_FACTOR: 3
      CONNECT_OFFSET_STORAGE_REPLICATION_FACTOR: 3
      CONNECT_STATUS_STORAGE_REPLICATION_FACTOR: 3
      CONNECT_PLUGIN_PATH: /usr/share/java,/usr/share/confluent-hub-components,/connectors
```

Start the updated environment:

```bash
docker-compose -f docker-compose-with-connect.yml up -d
```

## Step 10: Set Up Schema Registry

Schema Registry provides a serving layer for your metadata and ensures that Kafka messages follow a specific schema. Add Schema Registry to your Docker Compose file:

```yaml
version: '3'

services:
  # ... Previous services ...

  schema-registry:
    image: confluentinc/cp-schema-registry:7.3.0
    container_name: schema-registry
    depends_on:
      - kafka1
    ports:
      - "8081:8081"
    environment:
      SCHEMA_REGISTRY_HOST_NAME: schema-registry
      SCHEMA_REGISTRY_KAFKASTORE_BOOTSTRAP_SERVERS: kafka1:9092,kafka2:9092,kafka3:9092
      SCHEMA_REGISTRY_LISTENERS: http://0.0.0.0:8081
```

Start the updated environment:

```bash
docker-compose -f docker-compose-with-schema-registry.yml up -d
```

## Step 11: Set Up Kafka UI Tools

Kafka UI tools can help you manage and monitor your Kafka cluster. Add Kafka UI to your Docker Compose file:

```yaml
version: '3'

services:
  # ... Previous services ...

  kafka-ui:
    image: provectuslabs/kafka-ui:latest
    container_name: kafka-ui
    depends_on:
      - kafka1
      - kafka2
      - kafka3
      - schema-registry
    ports:
      - "8080:8080"
    environment:
      KAFKA_CLUSTERS_0_NAME: local
      KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: kafka1:9092,kafka2:9092,kafka3:9092
      KAFKA_CLUSTERS_0_ZOOKEEPER: zookeeper:2181
      KAFKA_CLUSTERS_0_SCHEMAREGISTRY: http://schema-registry:8081
```

Start the updated environment:

```bash
docker-compose -f docker-compose-with-ui.yml up -d
```

Access the Kafka UI at http://localhost:8080.

## Step 12: Configure Kafka for Production

For production environments, you should consider additional configurations:

### Security Configuration

Update your Docker Compose file to enable SSL and SASL authentication:

```yaml
version: '3'

services:
  # ... ZooKeeper configuration ...

  kafka1:
    # ... other configurations ...
    environment:
      # ... previous environment variables ...
      KAFKA_LISTENERS: SASL_SSL://kafka1:9092,SASL_SSL_HOST://localhost:9092
      KAFKA_ADVERTISED_LISTENERS: SASL_SSL://kafka1:9092,SASL_SSL_HOST://localhost:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: SASL_SSL:SASL_PLAINTEXT,SASL_SSL_HOST:SASL_PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: SASL_SSL
      KAFKA_SASL_MECHANISM_INTER_BROKER_PROTOCOL: PLAIN
      KAFKA_SASL_ENABLED_MECHANISMS: PLAIN
      KAFKA_OPTS: "-Djava.security.auth.login.config=/etc/kafka/kafka_server_jaas.conf"
    volumes:
      - ./kafka_server_jaas.conf:/etc/kafka/kafka_server_jaas.conf
      - ./kafka.keystore.jks:/etc/kafka/kafka.keystore.jks
      - ./kafka.truststore.jks:/etc/kafka/kafka.truststore.jks
```

Create the JAAS configuration file (`kafka_server_jaas.conf`):

```
KafkaServer {
  org.apache.kafka.common.security.plain.PlainLoginModule required
  username="admin"
  password="admin-secret"
  user_admin="admin-secret"
  user_producer="producer-secret"
  user_consumer="consumer-secret";
};
```

### Performance Tuning

Add performance-related configurations to your Kafka brokers:

```yaml
version: '3'

services:
  # ... ZooKeeper configuration ...

  kafka1:
    # ... other configurations ...
    environment:
      # ... previous environment variables ...
      KAFKA_NUM_PARTITIONS: 8
      KAFKA_DEFAULT_REPLICATION_FACTOR: 3
      KAFKA_LOG_RETENTION_HOURS: 168
      KAFKA_LOG_SEGMENT_BYTES: 1073741824
      KAFKA_LOG_RETENTION_CHECK_INTERVAL_MS: 300000
      KAFKA_NUM_NETWORK_THREADS: 5
      KAFKA_NUM_IO_THREADS: 8
      KAFKA_SOCKET_SEND_BUFFER_BYTES: 102400
      KAFKA_SOCKET_RECEIVE_BUFFER_BYTES: 102400
      KAFKA_SOCKET_REQUEST_MAX_BYTES: 104857600
      KAFKA_NUM_RECOVERY_THREADS_PER_DATA_DIR: 2
      KAFKA_LOG_FLUSH_INTERVAL_MESSAGES: 10000
      KAFKA_LOG_FLUSH_INTERVAL_MS: 1000
```

## Step 13: Monitoring Kafka with Prometheus and Grafana

Add monitoring to your Kafka cluster using Prometheus and Grafana:

```yaml
version: '3'

services:
  # ... Previous services ...

  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      GF_SECURITY_ADMIN_USER: admin
      GF_SECURITY_ADMIN_PASSWORD: admin
    volumes:
      - grafana-data:/var/lib/grafana

  kafka-exporter:
    image: danielqsj/kafka-exporter:latest
    container_name: kafka-exporter
    ports:
      - "9308:9308"
    command:
      - --kafka.server=kafka1:9092
      - --kafka.server=kafka2:9092
      - --kafka.server=kafka3:9092

volumes:
  grafana-data:
```

Create a `prometheus.yml` configuration file:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'kafka'
    static_configs:
      - targets: ['kafka-exporter:9308']
```

Start the monitoring services:

```bash
docker-compose -f docker-compose-monitoring.yml up -d
```

Access Prometheus at http://localhost:9090 and Grafana at http://localhost:3000.

## Step 14: Backup and Restore Kafka Data

### Backup Kafka Data

To back up Kafka data, you need to back up both ZooKeeper data and Kafka logs. Create a backup script:

```bash
#!/bin/bash

# Create backup directory
BACKUP_DIR="kafka_backup_$(date +%Y%m%d%H%M%S)"
mkdir -p $BACKUP_DIR

# Backup ZooKeeper data
docker exec -it zookeeper zkCli.sh -server localhost:2181 get / > $BACKUP_DIR/zookeeper_root.txt

# Backup Kafka topics configuration
docker exec -it kafka1 kafka-topics --list --bootstrap-server kafka1:9092 > $BACKUP_DIR/topics_list.txt

# For each topic, backup its configuration
while read topic; do
  docker exec -it kafka1 kafka-topics --describe --topic $topic --bootstrap-server kafka1:9092 > $BACKUP_DIR/topic_${topic}_config.txt
done < $BACKUP_DIR/topics_list.txt

# Backup Kafka data by copying the volume data
docker run --rm -v kafka_data:/data -v $(pwd)/$BACKUP_DIR:/backup alpine tar -czf /backup/kafka_data.tar.gz /data

echo "Backup completed in $BACKUP_DIR"
```

### Restore Kafka Data

To restore Kafka data, you need to restore both ZooKeeper data and Kafka logs:

```bash
#!/bin/bash

BACKUP_DIR=$1

if [ -z "$BACKUP_DIR" ]; then
  echo "Please provide the backup directory"
  exit 1
fi

# Stop Kafka and ZooKeeper
docker-compose down

# Restore Kafka data
docker volume rm kafka_data
docker volume create kafka_data
docker run --rm -v kafka_data:/data -v $(pwd)/$BACKUP_DIR:/backup alpine tar -xzf /backup/kafka_data.tar.gz -C /

# Start Kafka and ZooKeeper
docker-compose up -d

# Wait for Kafka to start
sleep 30

# Recreate topics if needed
while read topic; do
  config=$(grep -A 10 "Topic: $topic" $BACKUP_DIR/topic_${topic}_config.txt)
  partitions=$(echo "$config" | grep "PartitionCount" | awk '{print $2}')
  replication=$(echo "$config" | grep "ReplicationFactor" | awk '{print $2}')

  docker exec -it kafka1 kafka-topics --create --topic $topic --bootstrap-server kafka1:9092 --partitions $partitions --replication-factor $replication --if-not-exists
done < $BACKUP_DIR/topics_list.txt

echo "Restore completed from $BACKUP_DIR"
```

## Troubleshooting Common Issues

### Connection Issues

If you can't connect to Kafka:

1. Verify the containers are running:
   ```bash
   docker-compose ps
   ```

2. Check Kafka logs:
   ```bash
   docker logs kafka1
   ```

3. Verify network connectivity:
   ```bash
   docker exec -it kafka1 bash -c "ping zookeeper"
   ```

4. Check if Kafka is listening on the expected port:
   ```bash
   docker exec -it kafka1 netstat -tulpn | grep 9092
   ```

### Topic Creation Issues

If you can't create topics:

1. Check ZooKeeper connectivity:
   ```bash
   docker exec -it kafka1 bash -c "echo stat | nc zookeeper 2181"
   ```

2. Verify broker IDs are unique:
   ```bash
   docker exec -it kafka1 kafka-topics --describe --bootstrap-server kafka1:9092 --topic __consumer_offsets
   ```

### Performance Issues

If you're experiencing performance issues:

1. Check system resources:
   ```bash
   docker stats
   ```

2. Verify Kafka configurations:
   ```bash
   docker exec -it kafka1 cat /etc/kafka/server.properties
   ```

3. Monitor Kafka metrics using JMX:
   ```bash
   docker exec -it kafka1 kafka-run-class kafka.tools.JmxTool --jmx-url service:jmx:rmi:///jndi/rmi://localhost:9999/jmxrmi --object-name kafka.server:type=BrokerTopicMetrics,name=MessagesInPerSec
   ```

## Conclusion

Running Apache Kafka in Docker provides flexibility, consistency, and ease of deployment. By following this step-by-step guide, you can set up Kafka in Docker for various environments, from development to production.

Remember to follow security best practices, especially for production deployments, and regularly back up your data to prevent loss.

## References

- [Apache Kafka Documentation](https://kafka.apache.org/documentation/)
- [Confluent Docker Images](https://docs.confluent.io/platform/current/installation/docker/image-reference.html)
- [Docker Documentation](https://docs.docker.com/)
- [Kafka Security](https://kafka.apache.org/documentation/#security)
- [Kafka Monitoring](https://kafka.apache.org/documentation/#monitoring)