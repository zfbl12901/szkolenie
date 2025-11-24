# 1. Kubernetes Getting Started

## 🎯 Objectives

- Understand Kubernetes
- Install Kubernetes locally
- Understand basic concepts
- Create your first Pod

## 📋 Table of Contents

1. [Introduction to Kubernetes](#introduction-to-kubernetes)
2. [Installation](#installation)
3. [Basic Concepts](#basic-concepts)
4. [First Pods](#first-pods)
5. [Essential Commands](#essential-commands)

---

## Introduction to Kubernetes

### What is Kubernetes?

**Kubernetes (K8s)** = Container orchestration platform

- **Orchestration** : Manages multiple containers
- **Scaling** : Automatic scaling
- **Auto-healing** : Restarts failed containers
- **Load Balancing** : Traffic distribution
- **Rolling Updates** : Zero-downtime updates

### Why Kubernetes for Data Analyst?

- **Orchestration** : Manage multiple services
- **Scaling** : Adapt to needs
- **Resilience** : Auto-healing
- **Deployment** : Deploy easily

---

## Installation

### Minikube (recommended)

**Windows:**
```bash
# With Chocolatey
choco install minikube

# Start
minikube start

# Verify
kubectl get nodes
```

**Linux:**
```bash
# Install Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Start
minikube start

# Verify
kubectl get nodes
```

**macOS:**
```bash
# With Homebrew
brew install minikube

# Start
minikube start

# Verify
kubectl get nodes
```

### Install kubectl

**kubectl** = CLI for Kubernetes

**Windows:**
```bash
# With Chocolatey
choco install kubernetes-cli
```

**Linux:**
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

**macOS:**
```bash
# With Homebrew
brew install kubectl
```

---

## Basic Concepts

### Cluster

**Cluster** = Set of machines (nodes)

- **Master Node** : Manages the cluster
- **Worker Nodes** : Run containers

### Pod

**Pod** = Smallest deployable unit in Kubernetes

- **Containers** : One or more containers
- **Shared resources** : Network and storage
- **Ephemeral** : Can be created/destroyed

---

## First Pods

### Create a simple Pod

```bash
# Create a Pod
kubectl run nginx --image=nginx

# See Pods
kubectl get pods

# Describe a Pod
kubectl describe pod nginx

# Pod logs
kubectl logs nginx
```

### Pod with YAML

**nginx-pod.yaml:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

**Create the Pod:**
```bash
kubectl apply -f nginx-pod.yaml

# See the Pod
kubectl get pods

# Delete the Pod
kubectl delete -f nginx-pod.yaml
```

---

## Essential Commands

### Pod Management

```bash
# List Pods
kubectl get pods

# Describe a Pod
kubectl describe pod pod-name

# Pod logs
kubectl logs pod-name

# Execute a command in a Pod
kubectl exec -it pod-name -- bash

# Delete a Pod
kubectl delete pod pod-name
```

### Information

```bash
# See nodes
kubectl get nodes

# See namespaces
kubectl get namespaces

# Cluster information
kubectl cluster-info

# Version
kubectl version
```

---

## 📊 Key Takeaways

1. **Kubernetes** orchestrates containers
2. **Pods** are the smallest units
3. **kubectl** is the main CLI
4. **Minikube/Kind** for local Kubernetes
5. **YAML** to define resources

## 🔗 Next Module

Proceed to module [2. Fundamental Concepts](./02-concepts/README.md) to deepen.

