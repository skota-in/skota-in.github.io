# Angular Docker Guide

*Published on March 30, 2024*

## Overview

This guide provides a comprehensive overview of running Angular applications in Docker containers. It covers everything from basic concepts to advanced configurations, including development and production setups, multi-stage builds, environment configuration, and deployment strategies. Whether you're new to Angular or looking to optimize your existing Angular Docker setup, this guide will help you understand and implement best practices for containerized Angular deployments.

## Table of Contents

1. [What is Angular?](#what-is-angular)
2. [Getting Started](#getting-started)
3. [Basic Configuration](#basic-configuration)
4. [Development Setup](#development-setup)
5. [Production Setup](#production-setup)
6. [Environment Configuration](#environment-configuration)
7. [Multi-Stage Builds](#multi-stage-builds)
8. [Security](#security)
9. [Monitoring](#monitoring)
10. [Best Practices](#best-practices)
11. [References](#references)

## What is Angular?

Angular is a popular front-end framework that provides:

- Component-based architecture
- Two-way data binding
- Dependency injection
- Routing and navigation
- Forms handling
- HTTP client
- Testing utilities

### Key Features:
- TypeScript support
- Reactive programming
- Material Design components
- Progressive Web App support
- Internationalization
- Accessibility features

### Common Use Cases:
- Enterprise applications
- Single Page Applications
- Progressive Web Apps
- Mobile applications
- Real-time dashboards
- E-commerce platforms

## Getting Started

### Prerequisites
- Docker Engine (version 19.03 or later)
- Docker Compose (version 1.27 or later)
- Node.js (version 16 or later)
- Angular CLI (version 15 or later)
- Basic understanding of Angular concepts

### Verify Installation
```bash
# Check Docker version
docker --version

# Check Docker Compose version
docker-compose --version

# Check Node.js version
node --version

# Check Angular CLI version
ng --version
```

## Basic Configuration

### 1. Development Dockerfile
```dockerfile
# Stage 1: Build
FROM node:18-alpine as build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Serve
FROM nginx:alpine
COPY --from=build /app/dist/your-app-name /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### 2. Docker Compose Configuration
```yaml
version: '3.8'

services:
  angular:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: angular-app
    ports:
      - "4200:80"
    volumes:
      - ./src:/app/src
      - ./node_modules:/app/node_modules
    environment:
      - NODE_ENV=development
    restart: always
```

## Development Setup

### 1. Start Development Server
```bash
# Build and start container
docker-compose up -d

# View logs
docker-compose logs -f
```

### 2. Hot Reload Configuration
```yaml
services:
  angular:
    volumes:
      - ./src:/app/src
      - ./node_modules:/app/node_modules
    environment:
      - CHOKIDAR_USEPOLLING=true
      - WATCHPACK_POLLING=true
```

## Production Setup

### 1. Production Dockerfile
```dockerfile
# Stage 1: Build
FROM node:18-alpine as build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build --prod

# Stage 2: Serve
FROM nginx:alpine
COPY --from=build /app/dist/your-app-name /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### 2. Nginx Configuration
```nginx
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    # Enable gzip compression
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
}
```

## Environment Configuration

### 1. Environment Variables
```yaml
services:
  angular:
    environment:
      - API_URL=http://api.example.com
      - ENVIRONMENT=production
      - VERSION=1.0.0
```

### 2. Runtime Configuration
```typescript
// environment.ts
export const environment = {
  production: false,
  apiUrl: process.env.API_URL || 'http://localhost:3000',
  version: process.env.VERSION || '1.0.0'
};
```

## Multi-Stage Builds

### 1. Optimized Build
```dockerfile
# Stage 1: Build
FROM node:18-alpine as build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build --prod

# Stage 2: Test
FROM build as test
RUN npm run test

# Stage 3: Serve
FROM nginx:alpine
COPY --from=build /app/dist/your-app-name /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

## Security

### 1. Content Security Policy
```nginx
server {
    add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline';";
}
```

### 2. Security Headers
```nginx
server {
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Content-Type-Options "nosniff";
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains";
}
```

## Monitoring

### 1. Health Checks
```yaml
services:
  angular:
    healthcheck:
      test: ["CMD", "wget", "--spider", "http://localhost:80"]
      interval: 30s
      timeout: 10s
      retries: 3
```

### 2. Logging Configuration
```yaml
services:
  angular:
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

## Best Practices

1. **Configuration**:
   - Use multi-stage builds
   - Optimize Docker layers
   - Configure proper caching
   - Set appropriate timeouts
   - Use environment variables

2. **Security**:
   - Implement CSP headers
   - Use HTTPS
   - Regular security updates
   - Monitor dependencies
   - Follow Angular security guidelines

3. **Performance**:
   - Enable compression
   - Configure proper caching
   - Optimize assets
   - Use production mode
   - Implement lazy loading

4. **Maintenance**:
   - Regular updates
   - Monitor dependencies
   - Update configurations
   - Document procedures
   - Backup configurations

## References

- [Angular Documentation](https://angular.io/docs)
- [Docker Documentation](https://docs.docker.com/)
- [Nginx Documentation](https://nginx.org/en/docs/)
- [Angular Best Practices](https://angular.io/guide/styleguide)
- [Docker Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
