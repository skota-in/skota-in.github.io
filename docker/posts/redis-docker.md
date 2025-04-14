# Running Redis with Docker: A Step-by-Step Guide

*Published on August 20, 2023*

## Introduction

Redis is an open-source, in-memory data structure store used as a database, cache, message broker, and streaming engine. Running Redis in Docker containers offers numerous advantages, including simplified deployment, consistent environments, and easy version management.

This guide provides detailed, step-by-step instructions for setting up and running Redis using Docker, from basic setup to advanced configurations for production environments.

## Prerequisites

Before you begin, ensure you have the following installed:

- Docker Engine (version 19.03 or later)
- Docker Compose (version 1.27 or later) - optional but recommended

You can verify your Docker installation by running:

```bash
docker --version
docker-compose --version
```

## Step 1: Pull the Redis Docker Image

The first step is to pull the official Redis image from Docker Hub:

```bash
docker pull redis:7.0
```

This command downloads the Redis 7.0 image. You can specify a different version by changing the tag (e.g., `redis:6.2`, `redis:latest`).

## Step 2: Run a Basic Redis Container

To start a basic Redis container with default settings:

```bash
docker run --name redis -d -p 6379:6379 redis:7.0
```

This command:
- Creates a container named `redis`
- Runs it in detached mode (`-d`)
- Maps port 6379 from the container to port 6379 on your host
- Uses the Redis 7.0 image

## Step 3: Verify the Redis Container

Check if your Redis container is running:

```bash
docker ps
```

You should see your Redis container in the list of running containers.

To view the container logs:

```bash
docker logs redis
```

## Step 4: Connect to Redis

### Using the Redis CLI

You can connect to your Redis instance using the Redis CLI inside the container:

```bash
docker exec -it redis redis-cli
```

This opens an interactive Redis CLI where you can run commands like:

```
# Test the connection
PING

# Set a key-value pair
SET greeting "Hello, Docker!"

# Get a value
GET greeting

# List all keys
KEYS *

# Exit the CLI
EXIT
```

### Using a Redis Client

You can also connect to your Redis container using a Redis client like Redis Desktop Manager or another application. Use the following connection details:

- Host: localhost (or your Docker host IP)
- Port: 6379
- Password: none (unless configured)

## Step 5: Persist Redis Data

By default, data in Docker containers is ephemeral and will be lost when the container is removed. To persist your Redis data, you need to use Docker volumes:

```bash
docker run --name redis \
  -d \
  -p 6379:6379 \
  -v redis_data:/data \
  redis:7.0 redis-server --appendonly yes
```

This command:
- Creates a named volume `redis_data` that persists your Redis data
- Enables Redis Append Only File (AOF) persistence with `--appendonly yes`

To list your Docker volumes:

```bash
docker volume ls
```

## Step 6: Configure Redis with a Custom Configuration

For more advanced configurations, you can create a custom Redis configuration file:

1. Create a `redis.conf` file on your host machine:

```
# redis.conf
port 6379
bind 0.0.0.0
protected-mode yes
appendonly yes
appendfsync everysec
```

2. Run Redis with this configuration:

```bash
docker run --name redis \
  -d \
  -p 6379:6379 \
  -v $(pwd)/redis.conf:/usr/local/etc/redis/redis.conf \
  -v redis_data:/data \
  redis:7.0 redis-server /usr/local/etc/redis/redis.conf
```

This mounts your custom configuration file into the container and tells Redis to use it.

## Step 7: Enable Redis Authentication

To secure your Redis instance with a password:

```bash
docker run --name redis \
  -d \
  -p 6379:6379 \
  -v redis_data:/data \
  redis:7.0 redis-server --requirepass your_strong_password
```

To connect to a password-protected Redis instance:

```bash
docker exec -it redis redis-cli -a your_strong_password
```

Or within the Redis CLI:

```
AUTH your_strong_password
```

## Step 8: Set Up Redis with Docker Compose

Docker Compose simplifies managing multi-container applications. Create a `docker-compose.yml` file:

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
      - ./redis.conf:/usr/local/etc/redis/redis.conf
    command: redis-server /usr/local/etc/redis/redis.conf
    restart: always

volumes:
  redis_data:
```

To start Redis using Docker Compose:

```bash
docker-compose up -d
```

To stop it:

```bash
docker-compose down
```

## Step 9: Set Up Redis Sentinel for High Availability

For production environments, you might want to set up Redis Sentinel for high availability. Create a `docker-compose.yml` file for a Redis Sentinel setup:

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

  redis-sentinel-1:
    image: redis:7.0
    container_name: redis-sentinel-1
    ports:
      - "26379:26379"
    volumes:
      - ./sentinel.conf:/usr/local/etc/redis/sentinel.conf
    command: redis-sentinel /usr/local/etc/redis/sentinel.conf
    depends_on:
      - redis-master
      - redis-slave-1
      - redis-slave-2
    restart: always

  redis-sentinel-2:
    image: redis:7.0
    container_name: redis-sentinel-2
    ports:
      - "26380:26379"
    volumes:
      - ./sentinel.conf:/usr/local/etc/redis/sentinel.conf
    command: redis-sentinel /usr/local/etc/redis/sentinel.conf
    depends_on:
      - redis-master
      - redis-slave-1
      - redis-slave-2
    restart: always

  redis-sentinel-3:
    image: redis:7.0
    container_name: redis-sentinel-3
    ports:
      - "26381:26379"
    volumes:
      - ./sentinel.conf:/usr/local/etc/redis/sentinel.conf
    command: redis-sentinel /usr/local/etc/redis/sentinel.conf
    depends_on:
      - redis-master
      - redis-slave-1
      - redis-slave-2
    restart: always

volumes:
  redis_master_data:
  redis_slave_1_data:
  redis_slave_2_data:
```

Create a `sentinel.conf` file:

```
port 26379
sentinel monitor mymaster redis-master 6379 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 60000
sentinel parallel-syncs mymaster 1
```

Start the Redis Sentinel setup:

```bash
docker-compose up -d
```

## Step 10: Set Up Redis Cluster

For horizontal scaling, you can set up a Redis Cluster. Create a `docker-compose.yml` file for a 6-node Redis Cluster:

```yaml
version: '3.8'

services:
  redis-1:
    image: redis:7.0
    container_name: redis-1
    ports:
      - "6371:6379"
      - "16371:16379"
    volumes:
      - redis_1_data:/data
    command: redis-server --port 6379 --cluster-enabled yes --cluster-config-file nodes.conf --cluster-node-timeout 5000 --appendonly yes
    restart: always

  redis-2:
    image: redis:7.0
    container_name: redis-2
    ports:
      - "6372:6379"
      - "16372:16379"
    volumes:
      - redis_2_data:/data
    command: redis-server --port 6379 --cluster-enabled yes --cluster-config-file nodes.conf --cluster-node-timeout 5000 --appendonly yes
    restart: always

  redis-3:
    image: redis:7.0
    container_name: redis-3
    ports:
      - "6373:6379"
      - "16373:16379"
    volumes:
      - redis_3_data:/data
    command: redis-server --port 6379 --cluster-enabled yes --cluster-config-file nodes.conf --cluster-node-timeout 5000 --appendonly yes
    restart: always

  redis-4:
    image: redis:7.0
    container_name: redis-4
    ports:
      - "6374:6379"
      - "16374:16379"
    volumes:
      - redis_4_data:/data
    command: redis-server --port 6379 --cluster-enabled yes --cluster-config-file nodes.conf --cluster-node-timeout 5000 --appendonly yes
    restart: always

  redis-5:
    image: redis:7.0
    container_name: redis-5
    ports:
      - "6375:6379"
      - "16375:16379"
    volumes:
      - redis_5_data:/data
    command: redis-server --port 6379 --cluster-enabled yes --cluster-config-file nodes.conf --cluster-node-timeout 5000 --appendonly yes
    restart: always

  redis-6:
    image: redis:7.0
    container_name: redis-6
    ports:
      - "6376:6379"
      - "16376:16379"
    volumes:
      - redis_6_data:/data
    command: redis-server --port 6379 --cluster-enabled yes --cluster-config-file nodes.conf --cluster-node-timeout 5000 --appendonly yes
    restart: always

volumes:
  redis_1_data:
  redis_2_data:
  redis_3_data:
  redis_4_data:
  redis_5_data:
  redis_6_data:
```

Start the Redis Cluster containers:

```bash
docker-compose up -d
```

Create the Redis Cluster:

```bash
docker exec -it redis-1 redis-cli --cluster create \
  172.17.0.2:6379 172.17.0.3:6379 172.17.0.4:6379 \
  172.17.0.5:6379 172.17.0.6:6379 172.17.0.7:6379 \
  --cluster-replicas 1
```

Note: Replace the IP addresses with the actual IP addresses of your Redis containers. You can find them using:

```bash
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' redis-1 redis-2 redis-3 redis-4 redis-5 redis-6
```

## Step 11: Redis Monitoring and Management

### Using Redis Commander

Redis Commander is a web-based Redis management tool. Add it to your Docker Compose file:

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

  redis-commander:
    image: rediscommander/redis-commander:latest
    container_name: redis-commander
    ports:
      - "8081:8081"
    environment:
      - REDIS_HOSTS=local:redis:6379
    depends_on:
      - redis
    restart: always

volumes:
  redis_data:
```

Start the services:

```bash
docker-compose up -d
```

Access Redis Commander at http://localhost:8081.

## Step 12: Backup and Restore Redis Data

### Backup Redis Data

To create a backup of your Redis database:

```bash
docker exec redis redis-cli SAVE
docker cp redis:/data/dump.rdb ./backup_$(date +%Y%m%d%H%M%S).rdb
```

This creates a snapshot of your Redis database and copies it to your host machine with a timestamp in the filename.

### Restore Redis Data

To restore a backup to your Redis container:

```bash
docker cp ./backup_file.rdb redis:/data/dump.rdb
docker restart redis
```

## Step 13: Security Best Practices

### Network Security

Create a dedicated Docker network for your Redis container:

```bash
docker network create redis-network

docker run --name redis \
  -d \
  --network redis-network \
  -v redis_data:/data \
  redis:7.0 redis-server --appendonly yes
```

Only expose the Redis port to other containers in the same network, not to the host:

```yaml
version: '3.8'

services:
  redis:
    image: redis:7.0
    container_name: redis
    # No ports exposed to host
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes
    networks:
      - redis-network

  app:
    image: your-app-image
    container_name: app
    ports:
      - "3000:3000"
    depends_on:
      - redis
    networks:
      - redis-network

networks:
  redis-network:

volumes:
  redis_data:
```

### Redis Configuration Security

Secure your Redis configuration:

```
# redis.conf
port 6379
bind 0.0.0.0
protected-mode yes
requirepass your_strong_password
maxclients 10000
timeout 300
tcp-keepalive 300
appendonly yes
appendfsync everysec
```

### Use Docker Secrets

Use Docker secrets for sensitive information:

```yaml
version: '3.8'

services:
  redis:
    image: redis:7.0
    container_name: redis
    volumes:
      - redis_data:/data
      - ./redis.conf:/usr/local/etc/redis/redis.conf
    command: >
      bash -c "sed -i 's/\$\{REDIS_PASSWORD\}/'"$$(cat /run/secrets/redis_password)"'/g' /usr/local/etc/redis/redis.conf && \
               redis-server /usr/local/etc/redis/redis.conf"
    secrets:
      - redis_password

secrets:
  redis_password:
    file: ./secrets/redis_password.txt

volumes:
  redis_data:
```

In your `redis.conf`, use `requirepass ${REDIS_PASSWORD}` which will be replaced at runtime.

## Step 14: Redis Performance Tuning

Tune Redis for better performance:

```
# redis.conf

# Memory management
maxmemory 2gb
maxmemory-policy allkeys-lru

# Persistence
save 900 1
save 300 10
save 60 10000
appendonly yes
appendfsync everysec

# Clients
maxclients 10000
timeout 300

# Advanced settings
tcp-keepalive 300
no-appendfsync-on-rewrite yes
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```

## Troubleshooting Common Issues

### Container Won't Start

Check the container logs:

```bash
docker logs redis
```

Common issues include:
- Port conflicts: Change the host port mapping
- Permission issues with mounted volumes: Check volume permissions
- Configuration errors: Verify your Redis configuration file

### Connection Issues

If you can't connect to Redis:

1. Verify the container is running:
   ```bash
   docker ps | grep redis
   ```

2. Check if Redis is listening on the expected port:
   ```bash
   docker exec redis netstat -tulpn | grep 6379
   ```

3. Test connectivity from inside the container:
   ```bash
   docker exec -it redis redis-cli PING
   ```

### Memory Issues

If Redis is using too much memory or getting OOM (Out of Memory) errors:

1. Limit Redis memory usage:
   ```
   # redis.conf
   maxmemory 1gb
   maxmemory-policy allkeys-lru
   ```

2. Increase container memory limit:
   ```bash
   docker run --name redis -d -p 6379:6379 -m 2g redis:7.0
   ```

## Conclusion

Running Redis in Docker provides flexibility, consistency, and ease of deployment. By following this step-by-step guide, you can set up Redis in Docker for various environments, from development to production.

Remember to follow security best practices, especially for production deployments, and regularly back up your data to prevent loss.

## References

- [Redis Docker Hub Official Image](https://hub.docker.com/_/redis)
- [Redis Documentation](https://redis.io/documentation)
- [Docker Documentation](https://docs.docker.com/)
- [Redis Security](https://redis.io/topics/security)
- [Redis Persistence](https://redis.io/topics/persistence)
- [Redis High Availability](https://redis.io/topics/sentinel)
- [Redis Cluster](https://redis.io/topics/cluster-tutorial)