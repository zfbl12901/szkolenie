# 2. Kubernetes Fundamental Concepts

## 🎯 Objectives

- Understand Kubernetes architecture
- Master Pods, Nodes, Clusters
- Understand Controllers and ReplicaSets
- Use Namespaces

## 📋 Table of Contents

1. [Architecture](#architecture)
2. [Pods](#pods)
3. [Nodes](#nodes)
4. [Controllers](#controllers)
5. [Namespaces](#namespaces)

---

## Architecture

### Main Components

**Master Node:**
- **API Server** : Entry point
- **etcd** : Database
- **Scheduler** : Schedules Pods
- **Controller Manager** : Manages controllers

**Worker Node:**
- **kubelet** : Agent on each node
- **kube-proxy** : Network
- **Container Runtime** : Docker/containerd

---

## Pods

### What is a Pod?

**Pod** = Smallest deployable unit

- **One or more containers** : Share network/storage
- **Ephemeral** : Can be recreated
- **Unique IP** : Each Pod has an IP

### Pod Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: app
    image: nginx
    ports:
    - containerPort: 80
```

---

## Nodes

### Node Types

**Master Node:**
- Manages cluster
- Schedules Pods
- Manages state

**Worker Node:**
- Runs Pods
- Provides resources

---

## Controllers

### Controller Types

**ReplicaSet:**
- Maintains number of Pods
- Auto-healing

**Deployment:**
- Manages ReplicaSets
- Rolling updates

**StatefulSet:**
- For stateful applications
- Stable identity

---

## Namespaces

### Use Namespaces

```bash
# List namespaces
kubectl get namespaces

# Create a namespace
kubectl create namespace my-namespace

# Use a namespace
kubectl get pods -n my-namespace
```

---

## 📊 Key Takeaways

1. **Architecture** : Master and Worker Nodes
2. **Pods** : Smallest units
3. **Controllers** : Manage Pods
4. **Namespaces** : Logical isolation

## 🔗 Next Module

Proceed to module [3. Pods and Deployments](./03-pods-deployments/README.md).

