# 4. Dockerfile

## 🎯 Objectives

- Write a Dockerfile
- Understand instructions
- Optimize Dockerfiles
- Use multi-stage builds
- Best practices

## 📋 Table of Contents

1. [Introduction to Dockerfile](#introduction-to-dockerfile)
2. [Basic Instructions](#basic-instructions)
3. [Complete Dockerfile](#complete-dockerfile)
4. [Optimization](#optimization)
5. [Multi-stage Builds](#multi-stage-builds)

---

## Introduction to Dockerfile

### What is a Dockerfile?

**Dockerfile** = Instructions to build an image

- **Text** : Simple text file
- **Instructions** : Each line is an instruction
- **Automation** : Automates image creation
- **Versioned** : Can be versioned with Git

### Basic Structure

```dockerfile
# Comment
FROM base-image
RUN command
COPY source destination
CMD ["executable", "param"]
```

---

## Basic Instructions

### FROM

**Defines base image:**

```dockerfile
FROM python:3.11
FROM ubuntu:22.04
FROM alpine:latest
```

### WORKDIR

**Defines working directory:**

```dockerfile
WORKDIR /app
WORKDIR /usr/src/app
```

### COPY / ADD

**Copy files:**

```dockerfile
# COPY (recommended)
COPY requirements.txt .
COPY . .

# ADD (with automatic extraction)
ADD archive.tar.gz /app
```

### RUN

**Execute commands:**

```dockerfile
RUN apt update
RUN pip install -r requirements.txt

# Combine to reduce layers
RUN apt update && \
    apt install -y python3 && \
    apt clean
```

### CMD / ENTRYPOINT

**Default command:**

```dockerfile
# CMD (can be overridden)
CMD ["python", "app.py"]

# ENTRYPOINT (cannot be overridden)
ENTRYPOINT ["python"]
CMD ["app.py"]
```

### ENV

**Environment variables:**

```dockerfile
ENV PYTHONUNBUFFERED=1
ENV APP_ENV=production
```

### EXPOSE

**Expose ports:**

```dockerfile
EXPOSE 8080
EXPOSE 3306
```

---

## Complete Dockerfile

### Example 1 : Python Application

```dockerfile
# Base image
FROM python:3.11-slim

# Working directory
WORKDIR /app

# Environment variables
ENV PYTHONUNBUFFERED=1

# Copy requirements
COPY requirements.txt .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy code
COPY . .

# Expose port
EXPOSE 8000

# Default command
CMD ["python", "app.py"]
```

### Example 2 : Application with Data

```dockerfile
FROM python:3.11

WORKDIR /app

# Install system dependencies
RUN apt update && \
    apt install -y postgresql-client && \
    apt clean

# Install Python dependencies
COPY requirements.txt .
RUN pip install -r requirements.txt

# Copy application
COPY . .

# Create volume for data
VOLUME ["/app/data"]

# Expose port
EXPOSE 8080

# Command
CMD ["python", "main.py"]
```

---

## Optimization

### .dockerignore

**Create a `.dockerignore` file:**

```
__pycache__
*.pyc
.git
.env
node_modules
*.log
.DS_Store
```

### Reduce Layers

**Bad:**
```dockerfile
RUN apt update
RUN apt install -y python3
RUN apt install -y pip
RUN apt clean
```

**Good:**
```dockerfile
RUN apt update && \
    apt install -y python3 pip && \
    apt clean
```

### Instruction Order

**Put instructions that change little first:**

```dockerfile
# First dependencies (change little)
COPY requirements.txt .
RUN pip install -r requirements.txt

# Then code (changes often)
COPY . .
```

---

## Multi-stage Builds

### Why Multi-stage?

- **Reduce size** : Final image smaller
- **Security** : Exclude build tools
- **Performance** : Optimize builds

### Example

```dockerfile
# Stage 1 : Build
FROM python:3.11 as builder
WORKDIR /build
COPY requirements.txt .
RUN pip install --user -r requirements.txt

# Stage 2 : Runtime
FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY . .
ENV PATH=/root/.local/bin:$PATH
CMD ["python", "app.py"]
```

---

## 📊 Key Takeaways

1. **Dockerfile** automates image creation
2. **Instructions** : FROM, RUN, COPY, CMD
3. **Optimization** : Reduce layers
4. **Multi-stage** : Lighter images
5. **.dockerignore** : Exclude files

## 🔗 Next Module

Proceed to module [5. Docker Compose](./05-docker-compose/README.md) to orchestrate multiple containers.

