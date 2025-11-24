# 6. Docker Volumes and Networks

## 🎯 Objectives

- Understand Docker volumes
- Manage data persistence
- Create and manage networks
- Share data between containers
- Configure network communication

## 📋 Table of Contents

1. [Volumes](#volumes)
2. [Bind Mounts](#bind-mounts)
3. [Networks](#networks)
4. [Container Communication](#container-communication)
5. [Practical Examples](#practical-examples)

---

## Volumes

### What is a Volume?

**Volume** = Persistent storage for data

- **Persistent** : Survives container deletion
- **Managed by Docker** : Stored in `/var/lib/docker/volumes`
- **Shareable** : Multiple containers can use it
- **Performant** : Faster than bind mounts

### Create a Volume

```bash
# Create a volume
docker volume create my-volume

# List volumes
docker volume ls

# Inspect a volume
docker volume inspect my-volume

# Remove a volume
docker volume rm my-volume
```

### Use a Volume

```bash
# Named volume
docker run -v my-volume:/data ubuntu

# Anonymous volume
docker run -v /data ubuntu

# In docker-compose.yml
volumes:
  - my-volume:/data
```

---

## Bind Mounts

### What is a Bind Mount?

**Bind Mount** = Direct link to host directory

- **Direct** : Direct access to host files
- **Development** : Ideal for development
- **Performance** : Depends on host filesystem

### Use a Bind Mount

```bash
# Bind mount
docker run -v /host/path:/container/path ubuntu

# With docker-compose.yml
volumes:
  - ./data:/app/data
  - /absolute/path:/container/path
```

### Differences : Volume vs Bind Mount

**Volume:**
- Managed by Docker
- Better performance
- Portable
- Recommended for production

**Bind Mount:**
- Direct link
- Direct access
- Depends on host system
- Recommended for development

---

## Networks

### Network Types

1. **Bridge** : Default network (isolation)
2. **Host** : Uses host network
3. **None** : No network
4. **Overlay** : For Docker Swarm

### Create a Network

```bash
# Create a network
docker network create my-network

# List networks
docker network ls

# Inspect a network
docker network inspect my-network

# Remove a network
docker network rm my-network
```

### Connect a Container

```bash
# Connect on startup
docker run --network my-network ubuntu

# Connect existing container
docker network connect my-network container-id

# Disconnect
docker network disconnect my-network container-id
```

---

## Container Communication

### Same Network

```bash
# Create a network
docker network create app-network

# Container 1
docker run --name app --network app-network my-app

# Container 2 (can communicate with app)
docker run --name db --network app-network postgres
```

### With Docker Compose

```yaml
services:
  app:
    networks:
      - app-network
  
  db:
    networks:
      - app-network

networks:
  app-network:
```

### DNS Resolution

**Containers can find each other by name:**

```python
# In app container
import psycopg2
conn = psycopg2.connect(
    host="db",  # Service name
    database="mydb"
)
```

---

## Practical Examples

### Example 1 : Database with Volume

```yaml
version: '3.8'

services:
  db:
    image: postgres:15
    volumes:
      - db-data:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: mydb

volumes:
  db-data:
```

### Example 2 : Application with Bind Mount

```yaml
version: '3.8'

services:
  app:
    build: .
    volumes:
      - ./src:/app/src  # Development
    networks:
      - app-network

networks:
  app-network:
```

---

## 📊 Key Takeaways

1. **Volumes** for Docker-managed persistence
2. **Bind Mounts** for direct access
3. **Networks** for communication
4. **DNS** : Resolution by service name
5. **Docker Compose** simplifies management

## 🔗 Next Module

Proceed to module [7. Best Practices](./07-best-practices/README.md) for best practices.

