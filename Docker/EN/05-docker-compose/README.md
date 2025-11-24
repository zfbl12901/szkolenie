# 5. Docker Compose

## 🎯 Objectives

- Understand Docker Compose
- Orchestrate multiple containers
- Create docker-compose.yml files
- Manage services and networks
- Use environment variables

## 📋 Table of Contents

1. [Introduction to Docker Compose](#introduction-to-docker-compose)
2. [docker-compose.yml File](#docker-composeyml-file)
3. [Services](#services)
4. [Networks and Volumes](#networks-and-volumes)
5. [Commands](#commands)

---

## Introduction to Docker Compose

### What is Docker Compose?

**Docker Compose** = Tool to orchestrate multiple containers

- **Multi-containers** : Manages multiple containers
- **Configuration** : Simple YAML file
- **Orchestration** : Starts/stops all services
- **Networks** : Automatically creates networks

### Why Docker Compose?

- **Simplicity** : One file for everything
- **Reproducibility** : Same environment everywhere
- **Development** : Complete local stack
- **Production** : Simplified deployment

---

## docker-compose.yml File

### Basic Structure

```yaml
version: '3.8'

services:
  web:
    image: nginx
    ports:
      - "8080:80"
  
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: password
```

### Complete Example

```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8080:8000"
    environment:
      - DATABASE_URL=postgresql://db:5432/mydb
    depends_on:
      - db
  
  db:
    image: postgres:15
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

---

## Services

### Define a Service

```yaml
services:
  app:
    image: python:3.11
    command: python app.py
    working_dir: /app
    volumes:
      - .:/app
```

### Build vs Image

```yaml
# Use existing image
services:
  web:
    image: nginx:latest

# Build from Dockerfile
services:
  app:
    build: .
    # or
    build:
      context: .
      dockerfile: Dockerfile.prod
```

### Dependencies

```yaml
services:
  app:
    depends_on:
      - db
      - redis
  
  db:
    image: postgres
  
  redis:
    image: redis
```

---

## Networks and Volumes

### Networks

```yaml
services:
  app:
    networks:
      - frontend
      - backend
  
  db:
    networks:
      - backend

networks:
  frontend:
  backend:
```

### Volumes

```yaml
services:
  db:
    volumes:
      - db-data:/var/lib/postgresql/data
      - ./backup:/backup

volumes:
  db-data:
```

---

## Commands

### Start Services

```bash
# Start all services
docker-compose up

# In background
docker-compose up -d

# Rebuild images
docker-compose up --build
```

### Stop Services

```bash
# Stop services
docker-compose stop

# Stop and remove
docker-compose down

# Remove with volumes
docker-compose down -v
```

### Service Management

```bash
# See running services
docker-compose ps

# View logs
docker-compose logs

# Logs of a service
docker-compose logs web

# Execute a command
docker-compose exec web bash

# Restart a service
docker-compose restart web
```

---

## 📊 Key Takeaways

1. **Docker Compose** orchestrates multiple containers
2. **docker-compose.yml** defines configuration
3. **Services** are containers
4. **Networks and Volumes** for communication and data
5. **Commands** : up, down, logs, exec

## 🔗 Next Module

Proceed to module [6. Volumes and Networks](./06-volumes-networks/README.md) to deepen.

