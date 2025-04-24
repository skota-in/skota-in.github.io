# Dockerfile vs docker-compose.yml: A Comprehensive Guide

*Published on March 30, 2024*

## Overview

This guide explains the differences between Dockerfile and docker-compose.yml, their respective purposes, and when to use each. It covers basic concepts, practical examples, and best practices for both configuration files.

## Table of Contents

1. [What is a Dockerfile?](#what-is-a-dockerfile)
2. [What is docker-compose.yml?](#what-is-docker-composeyml)
3. [Key Differences](#key-differences)
4. [When to Use Each](#when-to-use-each)
5. [Practical Examples](#practical-examples)
6. [Best Practices](#best-practices)
7. [References](#references)

## What is a Dockerfile?

A Dockerfile is a text document that contains all the commands a user could call on the command line to assemble an image. It's used to create a single container image.

### Basic Structure:
```dockerfile
# Base image
FROM node:14-alpine

# Working directory
WORKDIR /app

# Copy files
COPY package*.json ./
COPY . .

# Install dependencies
RUN npm install

# Expose port
EXPOSE 3000

# Command to run
CMD ["npm", "start"]
```

### Key Features:
- Defines base image
- Sets up working directory
- Copies files
- Installs dependencies
- Configures environment
- Specifies commands to run

## What is docker-compose.yml?

docker-compose.yml is a YAML file that defines how Docker containers should behave in production. It's used to run multi-container Docker applications.

### Basic Structure:
```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    depends_on:
      - db

  db:
    image: postgres:13
    environment:
      - POSTGRES_PASSWORD=secret
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### Key Features:
- Defines multiple services
- Configures networking
- Sets up volumes
- Manages environment variables
- Handles dependencies
- Controls container lifecycle

## Key Differences

### 1. Purpose
- **Dockerfile**: Creates a single container image
- **docker-compose.yml**: Orchestrates multiple containers

### 2. Scope
- **Dockerfile**: Container-level configuration
- **docker-compose.yml**: Application-level configuration

### 3. Commands
- **Dockerfile**: Uses Docker-specific commands (FROM, RUN, COPY, etc.)
- **docker-compose.yml**: Uses YAML syntax for service definitions

### 4. Usage
- **Dockerfile**: `docker build -t image-name .`
- **docker-compose.yml**: `docker-compose up`

## When to Use Each

### Use Dockerfile When:
1. Creating a single container image
2. Defining the build process
3. Setting up the container environment
4. Creating reusable images
5. Building custom images

### Use docker-compose.yml When:
1. Running multiple containers
2. Managing container dependencies
3. Setting up development environments
4. Deploying multi-service applications
5. Managing container networks and volumes

## Practical Examples

### 1. Simple Web Application

#### Dockerfile:
```dockerfile
FROM nginx:alpine
COPY ./html /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

#### docker-compose.yml:
```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html
```

### 2. Full-Stack Application

#### Dockerfile (Backend):
```dockerfile
FROM openjdk:11-jdk
WORKDIR /app
COPY . .
RUN ./gradlew build
EXPOSE 8080
CMD ["java", "-jar", "build/libs/app.jar"]
```

#### Dockerfile (Frontend):
```dockerfile
FROM node:14-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
EXPOSE 3000
CMD ["npm", "start"]
```

#### docker-compose.yml:
```yaml
version: '3.8'

services:
  backend:
    build: ./backend
    ports:
      - "8080:8080"
    environment:
      - DB_URL=jdbc:postgresql://db:5432/app
    depends_on:
      - db

  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    environment:
      - API_URL=http://backend:8080
    depends_on:
      - backend

  db:
    image: postgres:13
    environment:
      - POSTGRES_DB=app
      - POSTGRES_PASSWORD=secret
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

## Best Practices

### 1. Dockerfile Best Practices:
- Use specific version tags
- Minimize layers
- Use .dockerignore
- Clean up after installations
- Use multi-stage builds
- Set proper working directory
- Use non-root users
- Handle signals properly

### 2. docker-compose.yml Best Practices:
- Use version 3 or higher
- Define proper service names
- Use environment variables
- Configure proper networks
- Set up volumes correctly
- Handle dependencies properly
- Use health checks
- Configure resource limits

### 3. General Best Practices:
- Keep configurations simple
- Use meaningful names
- Document configurations
- Version control both files
- Test configurations
- Use environment variables
- Implement proper logging
- Handle secrets securely

## References

- [Dockerfile Reference](https://docs.docker.com/engine/reference/builder/)
- [Docker Compose Reference](https://docs.docker.com/compose/compose-file/)
- [Docker Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Docker Compose Best Practices](https://docs.docker.com/compose/best-practices/)
