# 1. Docker Getting Started

## 🎯 Objectives

- Understand Docker and containerization
- Install Docker
- Understand basic concepts
- Run your first container

## 📋 Table of Contents

1. [Introduction to Docker](#introduction-to-docker)
2. [Installation](#installation)
3. [Basic Concepts](#basic-concepts)
4. [First Containers](#first-containers)
5. [Essential Commands](#essential-commands)

---

## Introduction to Docker

### What is Docker?

**Docker** = Containerization platform

- **Containers** : Isolated and lightweight environments
- **Portable** : Works everywhere (Windows, Linux, macOS)
- **Efficient** : Uses fewer resources than VMs
- **Fast** : Starts in seconds

### Why Docker for Data Analyst?

- **Reproducibility** : Same environment everywhere
- **Isolation** : Separate Python/R dependencies
- **Simplicity** : Easy to share and deploy
- **Performance** : Faster than VMs

---

## Installation

### Windows

1. Go to: https://www.docker.com/products/docker-desktop
2. Download Docker Desktop for Windows
3. Install the `.exe` file
4. Restart if needed

### Linux

```bash
# Update packages
sudo apt update

# Install dependencies
sudo apt install apt-transport-https ca-certificates curl gnupg lsb-release

# Add Docker GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Add Docker repository
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io

# Start Docker
sudo systemctl start docker
sudo systemctl enable docker

# Verify
docker --version
```

### macOS

1. Go to: https://www.docker.com/products/docker-desktop
2. Download Docker Desktop for Mac
3. Install the `.dmg` file
4. Open Docker from Applications

---

## Basic Concepts

### Images

**Image** = Read-only template for creating containers

- **Template** : Contains OS, applications, dependencies
- **Immutable** : Doesn't change once created
- **Lightweight** : Shares common layers

### Containers

**Container** = Executable instance of an image

- **Isolated** : Separate environment
- **Ephemeral** : Can be created/destroyed easily
- **Portable** : Works anywhere Docker is installed

---

## First Containers

### Hello World

```bash
# Run Hello World container
docker run hello-world
```

### Interactive Container

```bash
# Run interactive Ubuntu container
docker run -it ubuntu bash

# Inside container
ls
pwd
exit
```

### Background Container

```bash
# Run container in background
docker run -d --name my-container nginx

# See running containers
docker ps

# See logs
docker logs my-container

# Stop container
docker stop my-container
```

---

## Essential Commands

### Container Management

```bash
# List running containers
docker ps

# List all containers
docker ps -a

# Start container
docker start my-container

# Stop container
docker stop my-container

# Remove container
docker rm my-container
```

### Image Management

```bash
# List images
docker images

# Download image
docker pull ubuntu

# Remove image
docker rmi ubuntu
```

---

## 📊 Key Takeaways

1. **Docker = Containerization** to isolate applications
2. **Images** are templates, **Containers** are instances
3. **Docker Hub** to find images
4. **Basic commands** : run, ps, stop, rm
5. **Portable** : Works everywhere

## 🔗 Next Module

Proceed to module [2. Containers](./02-containers/README.md) to deepen container management.

