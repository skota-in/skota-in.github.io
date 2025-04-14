# Redis Docker Guide

*Published on March 30, 2024*

## Overview

This guide provides a comprehensive overview of running Redis in Docker containers. It covers everything from basic concepts to advanced configurations, including single-node and cluster setups, security configurations, monitoring, and production deployment. Whether you're new to Redis or looking to optimize your existing Redis Docker setup, this guide will help you understand and implement best practices for containerized Redis deployments.

## Table of Contents

1. [What is Redis?](#what-is-redis)
2. [Getting Started](#getting-started)
3. [Basic Configuration](#basic-configuration)
4. [Single-Node Setup](#single-node-setup)
5. [Redis Cluster](#redis-cluster)
6. [Security](#security)
7. [Monitoring](#monitoring)
8. [Production Deployment](#production-deployment)
9. [Best Practices](#best-practices)
10. [References](#references)

## What is Redis?

Redis is an open-source, in-memory data structure store that provides:

- In-memory data storage
- Data persistence options
- High performance
- Rich data types
- Pub/Sub messaging
- Lua scripting

### Key Features:
- In-memory storage
- Data persistence
- Replication
- High availability
- Clustering
- Security features

### Common Use Cases:
- Caching
- Session management
- Real-time analytics
- Message brokering
- Rate limiting
- Leaderboards

## Getting Started

### Prerequisites
- Docker Engine (version 19.03 or later)
- Docker Compose (version 1.27 or later)
- At least 2GB of RAM allocated to Docker
- Basic understanding of Redis concepts

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
  redis:
    image: redis:7.0
    container_name: redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes
    restart: always

volumes:
  redis_data:
```

### 2. Custom Configuration
```yaml
# redis.conf
port 6379
bind 0.0.0.0
protected-mode yes
appendonly yes
appendfsync everysec
maxmemory 1gb
maxmemory-policy allkeys-lru
```

## Single-Node Setup

### 1. Start Services
```bash
# Start container
docker-compose up -d

# Verify running container
docker-compose ps
```

### 2. Redis Operations
```bash
# Connect to Redis
docker exec -it redis redis-cli

# Test connection
PING

# Set key-value pair
SET greeting "Hello, Docker!"

# Get value
GET greeting
```

## Redis Cluster

### 1. Cluster Configuration
```yaml
version: '3.8'

services:
  redis-master:
    image: redis:7.0
    container_name: redis-master
    ports:
      - "6379:6379"
    volumes:
      - redis_master_data:/data
    command: redis-server --appendonly yes
    restart: always

  redis-slave-1:
    image: redis:7.0
    container_name: redis-slave-1
    ports:
      - "6380:6379"
    volumes:
      - redis_slave_1_data:/data
    command: redis-server --appendonly yes --slaveof redis-master 6379
    depends_on:
      - redis-master
    restart: always

  redis-slave-2:
    image: redis:7.0
    container_name: redis-slave-2
    ports:
      - "6381:6379"
    volumes:
      - redis_slave_2_data:/data
    command: redis-server --appendonly yes --slaveof redis-master 6379
    depends_on:
      - redis-master
    restart: always

volumes:
  redis_master_data:
  redis_slave_1_data:
  redis_slave_2_data:
```

### 2. Initialize Cluster
```bash
# Connect to master
docker exec -it redis-master redis-cli

# Check replication status
INFO replication
```

## Security

### 1. Authentication
```yaml
services:
  redis:
    command: redis-server --requirepass your_strong_password
    environment:
      - REDIS_PASSWORD=your_strong_password
```

### 2. Network Security
```yaml
services:
  redis:
    networks:
      - internal_network
    command: redis-server --bind 0.0.0.0 --protected-mode yes

networks:
  internal_network:
    internal: true
```

## Monitoring

### 1. Prometheus Configuration
```yaml
services:
  redis:
    environment:
      - REDIS_PASSWORD=your_strong_password
      - REDIS_METRICS_ENABLED=true
```

### 2. Health Checks
```yaml
services:
  redis:
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 30s
      timeout: 10s
      retries: 3
```

## Production Deployment

### 1. Resource Limits
```yaml
services:
  redis:
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
  redis:
    volumes:
      - redis_data:/data
      - redis_config:/usr/local/etc/redis
    environment:
      - REDIS_PASSWORD=your_strong_password
```

## Best Practices

1. **Configuration**:
   - Set appropriate memory limits
   - Configure persistence options
   - Enable protected mode
   - Set proper timeouts
   - Configure maxmemory policy

2. **Security**:
   - Enable authentication
   - Use network segmentation
   - Implement TLS encryption
   - Regular security updates
   - Monitor access logs

3. **Performance**:
   - Optimize memory settings
   - Configure proper persistence
   - Use appropriate data types
   - Monitor system metrics
   - Implement proper cleanup

4. **Maintenance**:
   - Regular backups
   - Monitor memory usage
   - Update configurations
   - Test failover scenarios
   - Document procedures

## References

- [Redis Documentation](https://redis.io/documentation)
- [Redis Docker Hub](https://hub.docker.com/_/redis)
- [Docker Compose Reference](https://docs.docker.com/compose/compose-file/)
- [Redis Best Practices](https://redis.io/topics/admin)
- [Redis Security](https://redis.io/topics/security)