# 2. Docker Containers

## 🎯 Objectives

- Create and manage containers
- Understand lifecycle
- Execute commands
- Manage logs
- Configure containers

## 📋 Table of Contents

1. [Container Lifecycle](#container-lifecycle)
2. [Create Containers](#create-containers)
3. [Execute Commands](#execute-commands)
4. [Logs and Debugging](#logs-and-debugging)
5. [Configuration](#configuration)

---

## Container Lifecycle

### Container States

1. **Created** : Container created but not started
2. **Running** : Container running
3. **Paused** : Container paused
4. **Stopped** : Container stopped
5. **Removed** : Container removed

### Lifecycle Commands

```bash
# Create a container
docker create --name my-container ubuntu

# Start a container
docker start my-container

# Stop a container
docker stop my-container

# Restart a container
docker restart my-container

# Pause
docker pause my-container

# Resume
docker unpause my-container

# Remove a container
docker rm my-container
```

---

## Create Containers

### Create with docker run

```bash
# Create and start a container
docker run ubuntu echo "Hello"

# Create without starting
docker create --name my-container ubuntu

# Create with custom name
docker run --name my-app ubuntu
```

### Important Options

```bash
# Interactive mode
docker run -it ubuntu bash

# Detached mode (background)
docker run -d nginx

# Expose a port
docker run -p 8080:80 nginx

# Mount a volume
docker run -v /host/path:/container/path ubuntu

# Environment variables
docker run -e MY_VAR=value ubuntu

# Container name
docker run --name my-container ubuntu
```

---

## Execute Commands

### Execute in Running Container

```bash
# Execute a command
docker exec my-container ls

# Interactive mode
docker exec -it my-container bash

# Execute Python
docker exec -it my-container python
```

### Execute on Startup

```bash
# Default command
docker run ubuntu echo "Hello"

# Override command
docker run ubuntu ls -la

# Execute a script
docker run -v $(pwd):/app ubuntu bash /app/script.sh
```

---

## Logs and Debugging

### View Logs

```bash
# Container logs
docker logs my-container

# Follow logs (tail -f)
docker logs -f my-container

# Last lines
docker logs --tail 100 my-container

# With timestamp
docker logs -t my-container
```

### Inspect a Container

```bash
# Complete information
docker inspect my-container

# Specific information
docker inspect --format='{{.State.Status}}' my-container

# Network configuration
docker inspect --format='{{.NetworkSettings.IPAddress}}' my-container
```

### Statistics

```bash
# Real-time statistics
docker stats

# Container statistics
docker stats my-container

# Statistics without streaming
docker stats --no-stream
```

---

## Configuration

### Environment Variables

```bash
# One variable
docker run -e MY_VAR=value ubuntu

# Multiple variables
docker run -e VAR1=value1 -e VAR2=value2 ubuntu

# .env file
docker run --env-file .env ubuntu
```

### Ports

```bash
# Expose a port
docker run -p 8080:80 nginx

# Expose multiple ports
docker run -p 8080:80 -p 3306:3306 my-app

# Expose all ports
docker run -P nginx
```

### Volumes

```bash
# Named volume
docker run -v my-volume:/data ubuntu

# Bind mount
docker run -v /host/path:/container/path ubuntu

# Anonymous volume
docker run -v /data ubuntu
```

---

## 📊 Key Takeaways

1. **Lifecycle** : Created → Running → Stopped → Removed
2. **docker run** : Creates and starts
3. **docker exec** : Executes in running container
4. **docker logs** : View logs
5. **Configuration** : Variables, ports, volumes

## 🔗 Next Module

Proceed to module [3. Docker Images](./03-images/README.md) to learn image management.

