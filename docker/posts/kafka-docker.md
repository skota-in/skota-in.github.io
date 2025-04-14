# Kafka Docker Guide

*Published on March 30, 2024*

## Overview

This guide provides a comprehensive overview of running Apache Kafka in Docker containers. It covers everything from basic concepts to advanced configurations, including single-node and multi-broker setups, security configurations, monitoring, and production deployment. Whether you're new to Kafka or looking to optimize your existing Kafka Docker setup, this guide will help you understand and implement best practices for containerized Kafka deployments.

## Table of Contents

1. [What is Kafka?](#what-is-kafka)
2. [Getting Started](#getting-started)
3. [Basic Configuration](#basic-configuration)
4. [Single-Node Setup](#single-node-setup)
5. [Multi-Broker Cluster](#multi-broker-cluster)
6. [Security](#security)
7. [Monitoring](#monitoring)
8. [Production Deployment](#production-deployment)
9. [Best Practices](#best-practices)
10. [References](#references)

## What is Kafka?

Apache Kafka is a distributed event streaming platform that provides:

- High-throughput message processing
- Fault-tolerant storage
- Real-time data streaming
- Scalable architecture
- Event sourcing capabilities

### Key Components:
- Brokers: Kafka servers that store and serve messages
- Topics: Categories or feeds for message publishing
- Partitions: Scalable units of topics
- Producers: Applications that send messages
- Consumers: Applications that read messages
- ZooKeeper: Coordination service (being phased out)

### Common Use Cases:
- Real-time data pipelines
- Event-driven architectures
- Stream processing
- Log aggregation
- Metrics collection
- Message queuing

## Getting Started

### Prerequisites
- Docker Engine (version 19.03 or later)
- Docker Compose (version 1.27 or later)
- At least 4GB of RAM allocated to Docker
- Basic understanding of Kafka concepts

### Verify Installation
```bash
# Check Docker version
docker --version

# Check Docker Compose version
docker-compose --version

# Verify Docker is running
docker info
```

## Basic Configuration

### 1. Single-Node Setup
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

### 2. Topic Management
```bash
# Create a topic
docker exec -it kafka kafka-topics --create \
  --topic test-topic \
  --bootstrap-server localhost:9092 \
  --partitions 1 \
  --replication-factor 1

# List topics
docker exec -it kafka kafka-topics --list \
  --bootstrap-server localhost:9092

# Describe topic
docker exec -it kafka kafka-topics --describe \
  --topic test-topic \
  --bootstrap-server localhost:9092
```

## Single-Node Setup

### 1. Start Services
```bash
# Start containers
docker-compose up -d

# Verify running containers
docker-compose ps
```

### 2. Test Message Flow
```bash
# Start producer
docker exec -it kafka kafka-console-producer \
  --topic test-topic \
  --bootstrap-server localhost:9092

# Start consumer
docker exec -it kafka kafka-console-consumer \
  --topic test-topic \
  --from-beginning \
  --bootstrap-server localhost:9092
```

## Multi-Broker Cluster

### 1. Cluster Configuration
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

### 2. Cluster Management
```bash
# Start cluster
docker-compose -f docker-compose-multi-broker.yml up -d

# Create replicated topic
docker exec -it kafka1 kafka-topics --create \
  --topic replicated-topic \
  --bootstrap-server kafka1:9092 \
  --partitions 3 \
  --replication-factor 3

# Test fault tolerance
docker stop kafka2
docker start kafka2
```

## Security

### 1. SSL Configuration
```yaml
services:
  kafka:
    environment:
      KAFKA_SSL_KEYSTORE_FILENAME: kafka.keystore.jks
      KAFKA_SSL_KEYSTORE_CREDENTIALS: keystore_password
      KAFKA_SSL_KEY_CREDENTIALS: key_password
      KAFKA_SSL_TRUSTSTORE_FILENAME: kafka.truststore.jks
      KAFKA_SSL_TRUSTSTORE_CREDENTIALS: truststore_password
      KAFKA_SECURITY_INTER_BROKER_PROTOCOL: SSL
      KAFKA_SSL_CLIENT_AUTH: required
```

### 2. SASL Authentication
```yaml
services:
  kafka:
    environment:
      KAFKA_SASL_ENABLED_MECHANISMS: PLAIN
      KAFKA_SASL_MECHANISM_INTER_BROKER_PROTOCOL: PLAIN
      KAFKA_SECURITY_INTER_BROKER_PROTOCOL: SASL_PLAINTEXT
```

## Monitoring

### 1. Prometheus Configuration
```yaml
services:
  kafka:
    environment:
      KAFKA_METRICS_REPORTERS: io.confluent.metrics.reporter.ConfluentMetricsReporter
      CONFLUENT_METRICS_REPORTER_BOOTSTRAP_SERVERS: kafka:9092
      CONFLUENT_METRICS_REPORTER_TOPIC_REPLICAS: 1
      CONFLUENT_METRICS_REPORTER_ZOOKEEPER_CONNECT: zookeeper:2181
```

### 2. Health Checks
```yaml
services:
  kafka:
    healthcheck:
      test: ["CMD", "kafka-topics", "--list", "--bootstrap-server", "localhost:9092"]
      interval: 30s
      timeout: 10s
      retries: 3
```

## Production Deployment

### 1. Resource Limits
```yaml
services:
  kafka:
    deploy:
      resources:
        limits:
          cpus: '4'
          memory: 8G
        reservations:
          cpus: '2'
          memory: 4G
```

### 2. Volume Management
```yaml
services:
  kafka:
    volumes:
      - kafka_data:/var/lib/kafka/data
      - kafka_config:/etc/kafka
    environment:
      KAFKA_LOG_DIRS: /var/lib/kafka/data
```

## Best Practices

1. **Configuration**:
   - Set appropriate broker settings
   - Configure proper replication
   - Enable compression
   - Set optimal timeouts
   - Configure proper cleanup

2. **Security**:
   - Enable SSL/TLS
   - Implement authentication
   - Use network segmentation
   - Regular security updates
   - Monitor access logs

3. **Performance**:
   - Optimize memory settings
   - Configure proper disk I/O
   - Use appropriate partitions
   - Monitor system metrics
   - Implement proper cleanup

4. **Maintenance**:
   - Regular backups
   - Monitor disk usage
   - Update configurations
   - Test failover scenarios
   - Document procedures

## References

- [Kafka Documentation](https://kafka.apache.org/documentation/)
- [Confluent Docker Images](https://hub.docker.com/r/confluentinc/cp-kafka)
- [Docker Compose Reference](https://docs.docker.com/compose/compose-file/)
- [Kafka Best Practices](https://docs.confluent.io/platform/current/kafka/deployment.html)
- [Kafka Security](https://docs.confluent.io/platform/current/security/index.html)