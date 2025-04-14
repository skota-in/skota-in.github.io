# MongoDB Docker Guide

*Published on March 30, 2024*

## Overview

This guide provides a comprehensive overview of running MongoDB in Docker containers. It covers everything from basic concepts to advanced configurations, including single-node and replica set setups, security configurations, monitoring, and production deployment. Whether you're new to MongoDB or looking to optimize your existing MongoDB Docker setup, this guide will help you understand and implement best practices for containerized MongoDB deployments.

## Table of Contents

1. [What is MongoDB?](#what-is-mongodb)
2. [Getting Started](#getting-started)
3. [Basic Configuration](#basic-configuration)
4. [Single-Node Setup](#single-node-setup)
5. [Replica Set](#replica-set)
6. [Security](#security)
7. [Monitoring](#monitoring)
8. [Production Deployment](#production-deployment)
9. [Best Practices](#best-practices)
10. [References](#references)

## What is MongoDB?

MongoDB is a NoSQL document database that provides:

- Flexible document model
- High performance
- High availability
- Horizontal scalability
- Rich query language

### Key Features:
- Document-based storage
- Indexing support
- Aggregation framework
- Replication
- Sharding
- Security features

### Common Use Cases:
- Content management
- Real-time analytics
- Mobile applications
- Internet of Things
- User data management
- Logging and event data

## Getting Started

### Prerequisites
- Docker Engine (version 19.03 or later)
- Docker Compose (version 1.27 or later)
- At least 2GB of RAM allocated to Docker
- Basic understanding of MongoDB concepts

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
version: '3.8'

services:
  mongodb:
    image: mongo:6.0
    container_name: mongodb
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password
    restart: always

volumes:
  mongodb_data:
```

### 2. Custom Configuration
```yaml
# mongod.conf
storage:
  dbPath: /data/db
systemLog:
  destination: file
  path: /var/log/mongodb/mongod.log
  logAppend: true
net:
  port: 27017
  bindIp: 0.0.0.0
security:
  authorization: enabled
```

## Single-Node Setup

### 1. Start Services
```bash
# Start container
docker-compose up -d

# Verify running container
docker-compose ps
```

### 2. Database Operations
```bash
# Connect to MongoDB
docker exec -it mongodb mongosh -u admin -p password

# Create database and collection
use mydb
db.users.insertOne({ name: "John Doe", email: "john@example.com" })

# Query data
db.users.find()
```

## Replica Set

### 1. Replica Set Configuration
```yaml
version: '3.8'

services:
  mongo1:
    image: mongo:6.0
    container_name: mongo1
    command: ["--replSet", "rs0", "--bind_ip_all"]
    ports:
      - "27017:27017"
    volumes:
      - mongo1_data:/data/db
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password
    restart: always

  mongo2:
    image: mongo:6.0
    container_name: mongo2
    command: ["--replSet", "rs0", "--bind_ip_all"]
    ports:
      - "27018:27017"
    volumes:
      - mongo2_data:/data/db
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password
    restart: always

  mongo3:
    image: mongo:6.0
    container_name: mongo3
    command: ["--replSet", "rs0", "--bind_ip_all"]
    ports:
      - "27019:27017"
    volumes:
      - mongo3_data:/data/db
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password
    restart: always

volumes:
  mongo1_data:
  mongo2_data:
  mongo3_data:
```

### 2. Initialize Replica Set
```bash
# Connect to primary node
docker exec -it mongo1 mongosh -u admin -p password

# Initialize replica set
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "mongo1:27017" },
    { _id: 1, host: "mongo2:27017" },
    { _id: 2, host: "mongo3:27017" }
  ]
})

# Check replica set status
rs.status()
```

## Security

### 1. Authentication
```yaml
services:
  mongodb:
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password
      - MONGO_INITDB_DATABASE=admin
```

### 2. Network Security
```yaml
services:
  mongodb:
    networks:
      - internal_network
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password
      - MONGO_INITDB_DATABASE=admin
      - MONGO_INITDB_ROOT_ROLE=root

networks:
  internal_network:
    internal: true
```

## Monitoring

### 1. Prometheus Configuration
```yaml
services:
  mongodb:
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password
      - MONGO_INITDB_DATABASE=admin
      - MONGO_INITDB_ROOT_ROLE=root
      - MONGO_INITDB_METRICS_ENABLED=true
```

### 2. Health Checks
```yaml
services:
  mongodb:
    healthcheck:
      test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
      interval: 30s
      timeout: 10s
      retries: 3
```

## Production Deployment

### 1. Resource Limits
```yaml
services:
  mongodb:
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 4G
        reservations:
          cpus: '1'
          memory: 2G
```

### 2. Volume Management
```yaml
services:
  mongodb:
    volumes:
      - mongodb_data:/data/db
      - mongodb_config:/data/configdb
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password
      - MONGO_INITDB_DATABASE=admin
```

## Best Practices

1. **Configuration**:
   - Use appropriate storage engine
   - Configure proper indexes
   - Set optimal memory limits
   - Enable journaling
   - Configure proper timeouts

2. **Security**:
   - Enable authentication
   - Use network segmentation
   - Implement role-based access
   - Regular security updates
   - Monitor access logs

3. **Performance**:
   - Optimize memory settings
   - Configure proper disk I/O
   - Use appropriate indexes
   - Monitor system metrics
   - Implement proper cleanup

4. **Maintenance**:
   - Regular backups
   - Monitor disk usage
   - Update configurations
   - Test failover scenarios
   - Document procedures

## References

- [MongoDB Documentation](https://docs.mongodb.com/)
- [MongoDB Docker Hub](https://hub.docker.com/_/mongo)
- [Docker Compose Reference](https://docs.docker.com/compose/compose-file/)
- [MongoDB Best Practices](https://docs.mongodb.com/manual/administration/production-notes/)
- [MongoDB Security](https://docs.mongodb.com/manual/security/)