# Spring Boot Docker Guide

*Published on March 30, 2024*

## Overview

This guide provides a comprehensive overview of containerizing Spring Boot applications using Docker. It covers everything from basic concepts to advanced deployment strategies, including multi-stage builds, environment configuration, security best practices, and production deployment. Whether you're new to Docker or looking to optimize your existing containerized Spring Boot applications, this guide will help you understand and implement best practices for containerization.

## Table of Contents

1. [What is Docker?](#what-is-docker)
2. [Getting Started](#getting-started)
3. [Basic Configuration](#basic-configuration)
4. [Dockerfile Creation](#dockerfile-creation)
5. [Image Optimization](#image-optimization)
6. [Environment Configuration](#environment-configuration)
7. [Docker Compose](#docker-compose)
8. [Security](#security)
9. [Monitoring](#monitoring)
10. [Production Deployment](#production-deployment)
11. [Best Practices](#best-practices)
12. [References](#references)

## What is Docker?

Docker is a platform for developing, shipping, and running applications in containers. It provides:

- Containerization technology
- Image management
- Container orchestration
- Network management
- Volume management

### Key Benefits:
- Consistent environments
- Isolation
- Portability
- Scalability
- Resource efficiency
- Simplified deployment

### Common Use Cases:
- Microservices
- CI/CD pipelines
- Development environments
- Production deployments
- Testing environments
- Cloud-native applications

## Getting Started

### Prerequisites
- Java Development Kit (JDK) 8 or later
- Maven or Gradle
- Docker Desktop (Windows/Mac) or Docker Engine (Linux)
- A Spring Boot application

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

### 1. Simple Dockerfile
```dockerfile
FROM openjdk:17-jdk-slim

WORKDIR /app

COPY target/*.jar app.jar

ENTRYPOINT ["java", "-jar", "app.jar"]
```

### 2. Spring Boot Application
```java
@SpringBootApplication
@RestController
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

    @GetMapping("/")
    public String hello() {
        return "Hello, Docker!";
    }
}
```

## Dockerfile Creation

### 1. Basic Dockerfile
```dockerfile
FROM openjdk:17-jdk-slim

WORKDIR /app

COPY target/*.jar app.jar

ENTRYPOINT ["java", "-jar", "app.jar"]
```

### 2. Multi-stage Build
```dockerfile
# Build stage
FROM maven:3.8.5-openjdk-17-slim AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package -DskipTests

# Run stage
FROM openjdk:17-jre-slim
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### 3. Production-ready Dockerfile
```dockerfile
# Build stage
FROM maven:3.8.5-openjdk-17-slim AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package -DskipTests

# Run stage
FROM openjdk:17-jre-slim
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar

# Add non-root user
RUN addgroup --system --gid 1001 appuser && \
    adduser --system --uid 1001 --gid 1001 appuser
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
    CMD curl -f http://localhost:8080/actuator/health || exit 1

ENTRYPOINT ["java", "-jar", "app.jar"]
```

## Image Optimization

### 1. Layer Caching
```dockerfile
FROM openjdk:17-jdk-slim

WORKDIR /app

# Copy dependencies first (changes less frequently)
COPY pom.xml .
RUN mvn dependency:go-offline

# Then copy source code and build (changes more frequently)
COPY src ./src
RUN mvn package

ENTRYPOINT ["java", "-jar", "target/app.jar"]
```

### 2. JVM Optimization
```dockerfile
FROM openjdk:17-jre-slim

WORKDIR /app
COPY target/*.jar app.jar

# JVM memory settings
ENV JAVA_OPTS="-Xms512m -Xmx512m -XX:+UseG1GC -XX:MaxGCPauseMillis=200"

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

## Environment Configuration

### 1. Environment Variables
```bash
docker run -p 8080:8080 \
  -e SPRING_PROFILES_ACTIVE=prod \
  -e SPRING_DATASOURCE_URL=jdbc:mysql://db:3306/mydb \
  -e SPRING_DATASOURCE_USERNAME=user \
  -e SPRING_DATASOURCE_PASSWORD=pass \
  springboot-app:1.0
```

### 2. Docker Secrets
```bash
# Create secrets
echo "my-secret-password" | docker secret create db_password -
echo "my-secret-key" | docker secret create app_key -

# Use secrets in Docker service
docker service create \
  --name springboot-app \
  --secret db_password \
  --secret app_key \
  --env SPRING_DATASOURCE_PASSWORD_FILE=/run/secrets/db_password \
  --env APP_KEY_FILE=/run/secrets/app_key \
  springboot-app:1.0
```

## Docker Compose

### 1. Basic Compose File
```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=dev
    depends_on:
      - db

  db:
    image: mysql:8.0
    environment:
      - MYSQL_ROOT_PASSWORD=root
      - MYSQL_DATABASE=mydb
    volumes:
      - mysql_data:/var/lib/mysql

volumes:
  mysql_data:
```

### 2. Production Compose File
```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=prod
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
    depends_on:
      - db
    networks:
      - app-network

  db:
    image: mysql:8.0
    environment:
      - MYSQL_ROOT_PASSWORD_FILE=/run/secrets/db_root_password
      - MYSQL_DATABASE=mydb
    volumes:
      - mysql_data:/var/lib/mysql
    secrets:
      - db_root_password
    networks:
      - app-network

networks:
  app-network:
    driver: overlay

volumes:
  mysql_data:

secrets:
  db_root_password:
    file: ./secrets/db_root_password.txt
```

## Security

### 1. Non-root User
```dockerfile
FROM openjdk:17-jre-slim

WORKDIR /app
COPY target/*.jar app.jar

# Add non-root user
RUN addgroup --system --gid 1001 appuser && \
    adduser --system --uid 1001 --gid 1001 appuser

# Set permissions
RUN chown -R appuser:appuser /app

# Switch to non-root user
USER appuser

ENTRYPOINT ["java", "-jar", "app.jar"]
```

### 2. Security Headers
```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .headers()
                .contentSecurityPolicy("default-src 'self'")
                .and()
                .frameOptions().deny()
                .and()
                .xssProtection()
                .and()
                .httpStrictTransportSecurity();
    }
}
```

## Monitoring

### 1. Health Checks
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
    CMD curl -f http://localhost:8080/actuator/health || exit 1
```

### 2. Prometheus Configuration
```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,prometheus
  metrics:
    export:
      prometheus:
        enabled: true
```

## Production Deployment

### 1. Docker Swarm
```bash
# Initialize swarm
docker swarm init

# Deploy stack
docker stack deploy -c docker-compose.yml myapp
```

### 2. Kubernetes
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: springboot-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: springboot-app
  template:
    metadata:
      labels:
        app: springboot-app
    spec:
      containers:
      - name: springboot-app
        image: springboot-app:1.0
        ports:
        - containerPort: 8080
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: "prod"
```

## Best Practices

1. **Image Management**:
   - Use multi-stage builds
   - Minimize image size
   - Use specific version tags
   - Remove unnecessary files
   - Use .dockerignore

2. **Security**:
   - Run as non-root user
   - Use secrets for sensitive data
   - Keep images updated
   - Scan for vulnerabilities
   - Implement security headers

3. **Performance**:
   - Optimize JVM settings
   - Use appropriate base images
   - Implement health checks
   - Configure resource limits
   - Use caching effectively

4. **Maintenance**:
   - Document configurations
   - Version control Dockerfiles
   - Implement CI/CD
   - Monitor container health
   - Regular updates

## References

- [Docker Documentation](https://docs.docker.com/)
- [Spring Boot Docker Guide](https://spring.io/guides/gs/spring-boot-docker/)
- [Docker Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Spring Boot Actuator](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html)
- [Docker Security](https://docs.docker.com/engine/security/)
