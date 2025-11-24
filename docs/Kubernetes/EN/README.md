# Kubernetes Training for Data Analyst

## 📚 Overview

This training guides you through learning **Kubernetes** as a Data Analyst. Kubernetes is an open-source platform for orchestrating and managing containers at scale.

## 🎯 Learning Objectives

- Understand Kubernetes and container orchestration
- Install Kubernetes (locally)
- Create and manage Pods and Deployments
- Configure Services and Ingress
- Manage ConfigMaps and Secrets
- Use Persistent Volumes
- Create practical projects for your portfolio

## 💰 Everything is Free!

This training uses only:
- ✅ **Minikube / Kind** : Free local Kubernetes
- ✅ **kubectl** : Free Kubernetes CLI
- ✅ **Official Documentation** : Complete free guides
- ✅ **Online Tutorials** : Free resources

**Total Budget: $0**

## 📖 Training Structure

### 1. [Kubernetes Getting Started](./01-getting-started/README.md)
   - Install Kubernetes locally
   - Basic concepts
   - First Pods
   - Essential commands

### 2. [Fundamental Concepts](./02-concepts/README.md)
   - Kubernetes architecture
   - Pods, Nodes, Clusters
   - Controllers and ReplicaSets
   - Namespaces

### 3. [Pods and Deployments](./03-pods-deployments/README.md)
   - Create Pods
   - Manage Deployments
   - Scaling and Rolling Updates
   - Health Checks

### 4. [Services](./04-services/README.md)
   - Service types
   - Service Discovery
   - Load Balancing
   - Ingress

### 5. [ConfigMaps and Secrets](./05-configmaps-secrets/README.md)
   - Manage configuration
   - Secrets management
   - Environment variables
   - Best practices

### 6. [Persistent Volumes](./06-persistent-volumes/README.md)
   - Kubernetes Volumes
   - Persistent Volumes
   - Storage Classes
   - StatefulSets

### 7. [Best Practices](./07-best-practices/README.md)
   - Security
   - Performance
   - Organization
   - Monitoring

### 8. [Practical Projects](./08-projets/README.md)
   - Deploy a web application
   - Data pipeline with Kubernetes
   - Complete stack
   - Portfolio projects

## 🚀 Quick Start

### Prerequisites

- **Docker** : Installed and working
- **4 GB RAM** : Minimum recommended
- **Disk Space** : 20 GB free

### Quick Installation

**Minikube (recommended):**

```bash
# Install Minikube
# Windows
choco install minikube

# Linux
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# macOS
brew install minikube

# Start Minikube
minikube start

# Verify
kubectl get nodes
```

### First Pod

```bash
# Create a Pod
kubectl run nginx --image=nginx

# See Pods
kubectl get pods

# Describe a Pod
kubectl describe pod nginx
```

## 📊 Use Cases for Data Analyst

- **Orchestration** : Manage multiple containers
- **Scaling** : Scale automatically
- **Deployment** : Deploy easily
- **Resilience** : Auto-healing
- **Data Pipelines** : Orchestrate pipelines

## 📚 Free Resources

### Official Documentation

- **Kubernetes Documentation** : https://kubernetes.io/docs/
- **Kubernetes Playground** : https://www.katacoda.com/courses/kubernetes
- **Minikube** : https://minikube.sigs.k8s.io/

