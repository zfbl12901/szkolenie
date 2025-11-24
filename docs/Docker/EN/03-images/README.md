# 3. Docker Images

## 🎯 Objectives

- Understand Docker images
- Download images
- Create custom images
- Manage images
- Optimize images

## 📋 Table of Contents

1. [Introduction to Images](#introduction-to-images)
2. [Download Images](#download-images)
3. [Create Images](#create-images)
4. [Manage Images](#manage-images)
5. [Optimization](#optimization)

---

## Introduction to Images

### What is an Image?

**Image** = Read-only template

- **Template** : For creating containers
- **Layering** : Composed of layers
- **Immutable** : Doesn't change
- **Shared** : Multiple containers can use the same image

### Image Structure

```
Image
├── Layer 1 : Base OS (Ubuntu)
├── Layer 2 : System tools
├── Layer 3 : Python
└── Layer 4 : Your application
```

---

## Download Images

### Docker Hub

**Docker Hub** = Public image registry

- **Official images** : python, postgres, nginx, etc.
- **Community images** : Created by community
- **Free** : For public use

### Download an Image

```bash
# Download an image
docker pull python:3.11

# Download latest version
docker pull python:latest

# Download specific version
docker pull python:3.11-slim

# Search images
docker search python
```

### Popular Images for Data Analyst

```bash
# Python
docker pull python:3.11

# Jupyter Notebook
docker pull jupyter/scipy-notebook

# PostgreSQL
docker pull postgres:15

# MySQL
docker pull mysql:8.0

# Redis
docker pull redis:7
```

---

## Create Images

### With Dockerfile

**Create a Dockerfile:**

```dockerfile
# Dockerfile
FROM python:3.11

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

CMD ["python", "app.py"]
```

**Build the image:**

```bash
# Build an image
docker build -t my-app:latest .

# With specific tag
docker build -t my-app:v1.0 .

# From specific Dockerfile
docker build -f Dockerfile.prod -t my-app:prod .
```

### Commit from Container

```bash
# Create a container
docker run -it ubuntu bash

# Make modifications in container
apt update
apt install python3

# Create image from container
docker commit container-id my-image:tag
```

---

## Manage Images

### List Images

```bash
# List all images
docker images

# Filter by name
docker images python

# Show only IDs
docker images -q
```

### Remove Images

```bash
# Remove an image
docker rmi my-image:tag

# Remove by ID
docker rmi image-id

# Remove unused images
docker image prune

# Remove all images
docker rmi $(docker images -q)
```

### Tag Images

```bash
# Create a tag
docker tag my-image:latest my-image:v1.0

# Tag for Docker Hub
docker tag my-image:latest username/my-image:latest
```

### Inspect an Image

```bash
# Complete information
docker inspect my-image

# Layer history
docker history my-image

# Image size
docker images my-image
```

---

## Optimization

### Lightweight Images

**Use slim images:**

```dockerfile
# Instead of
FROM python:3.11

# Use
FROM python:3.11-slim
```

**Multi-stage builds:**

```dockerfile
# Stage 1 : Build
FROM python:3.11 as builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user -r requirements.txt

# Stage 2 : Runtime
FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY . .
CMD ["python", "app.py"]
```

### Reduce Size

**Best practices:**
1. Use `.dockerignore`
2. Combine RUN commands
3. Use lightweight base images
4. Clean caches

---

## 📊 Key Takeaways

1. **Images** are templates for containers
2. **Docker Hub** to find images
3. **Dockerfile** to create images
4. **Layering** enables sharing
5. **Optimization** reduces size

## 🔗 Next Module

Proceed to module [4. Dockerfile](./04-dockerfile/README.md) to learn writing Dockerfiles.

