# Running MongoDB with Docker: A Step-by-Step Guide

*Published on August 15, 2023*

## Introduction

MongoDB is a popular NoSQL database that stores data in flexible, JSON-like documents. Running MongoDB in Docker containers offers numerous advantages, including simplified deployment, consistent environments across development and production, and easy version management.

This guide provides detailed, step-by-step instructions for setting up and running MongoDB using Docker, from basic setup to advanced configurations for production environments.

## Prerequisites

Before you begin, ensure you have the following installed:

- Docker Engine (version 19.03 or later)
- Docker Compose (version 1.27 or later) - optional but recommended

You can verify your Docker installation by running:

```bash
docker --version
docker-compose --version
```

## Step 1: Pull the MongoDB Docker Image

The first step is to pull the official MongoDB image from Docker Hub:

```bash
docker pull mongo:6.0
```

This command downloads the MongoDB 6.0 image. You can specify a different version by changing the tag (e.g., `mongo:5.0`, `mongo:latest`).

## Step 2: Run a Basic MongoDB Container

To start a basic MongoDB container with default settings:

```bash
docker run --name mongodb -d -p 27017:27017 mongo:6.0
```

This command:
- Creates a container named `mongodb`
- Runs it in detached mode (`-d`)
- Maps port 27017 from the container to port 27017 on your host
- Uses the MongoDB 6.0 image

## Step 3: Verify the MongoDB Container

Check if your MongoDB container is running:

```bash
docker ps
```

You should see your MongoDB container in the list of running containers.

To view the container logs:

```bash
docker logs mongodb
```

## Step 4: Connect to MongoDB

### Using the MongoDB Shell

You can connect to your MongoDB instance using the MongoDB shell inside the container:

```bash
docker exec -it mongodb mongosh
```

This opens an interactive MongoDB shell where you can run commands like:

```javascript
// Show all databases
show dbs

// Create and use a new database
use mydb

// Create a collection and insert a document
db.users.insertOne({ name: "John Doe", email: "john@example.com" })

// Find documents
db.users.find()
```

### Using a MongoDB Client

You can also connect to your MongoDB container using a MongoDB client like MongoDB Compass. Use the following connection string:

```
mongodb://localhost:27017
```

## Step 5: Persist MongoDB Data

By default, data in Docker containers is ephemeral and will be lost when the container is removed. To persist your MongoDB data, you need to use Docker volumes:

```bash
docker run --name mongodb \
  -d \
  -p 27017:27017 \
  -v mongodb_data:/data/db \
  mongo:6.0
```

This command creates a named volume `mongodb_data` that persists your database data even if the container is removed.

To list your Docker volumes:

```bash
docker volume ls
```

## Step 6: Configure MongoDB with Environment Variables

You can configure MongoDB using environment variables:

```bash
docker run --name mongodb \
  -d \
  -p 27017:27017 \
  -v mongodb_data:/data/db \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=password \
  mongo:6.0
```

This creates a MongoDB instance with authentication enabled. To connect to this instance:

```bash
docker exec -it mongodb mongosh -u admin -p password
```

## Step 7: Create a Custom MongoDB Configuration

For more advanced configurations, you can create a custom MongoDB configuration file:

1. Create a `mongod.conf` file on your host machine:

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

2. Run MongoDB with this configuration:

```bash
docker run --name mongodb \
  -d \
  -p 27017:27017 \
  -v mongodb_data:/data/db \
  -v $(pwd)/mongod.conf:/etc/mongod.conf \
  -v mongodb_logs:/var/log/mongodb \
  mongo:6.0 --config /etc/mongod.conf
```

This mounts your custom configuration file into the container and tells MongoDB to use it.

## Step 8: Set Up MongoDB with Docker Compose

Docker Compose simplifies managing multi-container applications. Create a `docker-compose.yml` file:

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
      - ./mongod.conf:/etc/mongod.conf
      - mongodb_logs:/var/log/mongodb
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password
    command: ["--config", "/etc/mongod.conf"]
    restart: always

volumes:
  mongodb_data:
  mongodb_logs:
```

To start MongoDB using Docker Compose:

```bash
docker-compose up -d
```

To stop it:

```bash
docker-compose down
```

## Step 9: Set Up MongoDB Replica Set

For production environments, you might want to set up a MongoDB replica set for high availability. Create a `docker-compose.yml` file for a three-node replica set:

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
    restart: always

  mongo2:
    image: mongo:6.0
    container_name: mongo2
    command: ["--replSet", "rs0", "--bind_ip_all"]
    ports:
      - "27018:27017"
    volumes:
      - mongo2_data:/data/db
    restart: always

  mongo3:
    image: mongo:6.0
    container_name: mongo3
    command: ["--replSet", "rs0", "--bind_ip_all"]
    ports:
      - "27019:27017"
    volumes:
      - mongo3_data:/data/db
    restart: always

volumes:
  mongo1_data:
  mongo2_data:
  mongo3_data:
```

Start the replica set:

```bash
docker-compose up -d
```

Initialize the replica set:

```bash
docker exec -it mongo1 mongosh --eval "rs.initiate({_id: 'rs0', members: [{_id: 0, host: 'mongo1:27017'}, {_id: 1, host: 'mongo2:27017'}, {_id: 2, host: 'mongo3:27017'}]})"
```

Check the replica set status:

```bash
docker exec -it mongo1 mongosh --eval "rs.status()"
```

## Step 10: Backup and Restore MongoDB Data

### Backup MongoDB Data

To create a backup of your MongoDB database:

```bash
docker exec -it mongodb mongodump --out /data/backup
```

Copy the backup from the container to your host:

```bash
docker cp mongodb:/data/backup ./backup
```

### Restore MongoDB Data

To restore a backup to your MongoDB container:

1. Copy the backup to the container:

```bash
docker cp ./backup mongodb:/data/backup
```

2. Restore the backup:

```bash
docker exec -it mongodb mongorestore /data/backup
```

## Step 11: MongoDB Monitoring and Management

### Using MongoDB Express

MongoDB Express is a web-based MongoDB admin interface. Add it to your Docker Compose file:

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

  mongo-express:
    image: mongo-express
    container_name: mongo-express
    ports:
      - "8081:8081"
    environment:
      - ME_CONFIG_MONGODB_ADMINUSERNAME=admin
      - ME_CONFIG_MONGODB_ADMINPASSWORD=password
      - ME_CONFIG_MONGODB_SERVER=mongodb
    depends_on:
      - mongodb
    restart: always

volumes:
  mongodb_data:
```

Start the services:

```bash
docker-compose up -d
```

Access MongoDB Express at http://localhost:8081.

## Step 12: Security Best Practices

### Network Security

Create a dedicated Docker network for your MongoDB container:

```bash
docker network create mongodb-network

docker run --name mongodb \
  -d \
  --network mongodb-network \
  -v mongodb_data:/data/db \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=password \
  mongo:6.0
```

Only expose the MongoDB port to other containers in the same network, not to the host:

```yaml
version: '3.8'

services:
  mongodb:
    image: mongo:6.0
    container_name: mongodb
    # No ports exposed to host
    volumes:
      - mongodb_data:/data/db
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password
    networks:
      - mongodb-network

  app:
    image: your-app-image
    container_name: app
    ports:
      - "3000:3000"
    depends_on:
      - mongodb
    networks:
      - mongodb-network

networks:
  mongodb-network:

volumes:
  mongodb_data:
```

### Authentication and Authorization

Create a dedicated user for your application with restricted permissions:

```javascript
use admin
db.createUser({
  user: "appuser",
  pwd: "apppassword",
  roles: [{ role: "readWrite", db: "myapp" }]
})
```

### Secure Configuration

Use Docker secrets for sensitive information:

```yaml
version: '3.8'

services:
  mongodb:
    image: mongo:6.0
    container_name: mongodb
    volumes:
      - mongodb_data:/data/db
    environment:
      - MONGO_INITDB_ROOT_USERNAME_FILE=/run/secrets/mongo_root_username
      - MONGO_INITDB_ROOT_PASSWORD_FILE=/run/secrets/mongo_root_password
    secrets:
      - mongo_root_username
      - mongo_root_password

secrets:
  mongo_root_username:
    file: ./secrets/mongo_root_username.txt
  mongo_root_password:
    file: ./secrets/mongo_root_password.txt

volumes:
  mongodb_data:
```

## Troubleshooting Common Issues

### Container Won't Start

Check the container logs:

```bash
docker logs mongodb
```

Common issues include:
- Port conflicts: Change the host port mapping
- Permission issues with mounted volumes: Check volume permissions
- Configuration errors: Verify your MongoDB configuration file

### Connection Issues

If you can't connect to MongoDB:

1. Verify the container is running:
   ```bash
   docker ps | grep mongodb
   ```

2. Check if MongoDB is listening on the expected port:
   ```bash
   docker exec mongodb netstat -tulpn | grep 27017
   ```

3. Test connectivity from inside the container:
   ```bash
   docker exec -it mongodb mongosh
   ```

### Data Persistence Issues

If your data is not persisting between container restarts:

1. Check if you're using volumes correctly:
   ```bash
   docker inspect mongodb | grep -A 10 Mounts
   ```

2. Verify the volume exists and contains data:
   ```bash
   docker volume inspect mongodb_data
   ```

## Conclusion

Running MongoDB in Docker provides flexibility, consistency, and ease of deployment. By following this step-by-step guide, you can set up MongoDB in Docker for various environments, from development to production.

Remember to follow security best practices, especially for production deployments, and regularly back up your data to prevent loss.

## References

- [MongoDB Docker Hub Official Image](https://hub.docker.com/_/mongo)
- [MongoDB Documentation](https://docs.mongodb.com/)
- [Docker Documentation](https://docs.docker.com/)
- [MongoDB Security Checklist](https://docs.mongodb.com/manual/administration/security-checklist/)