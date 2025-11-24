# 4. Kubernetes Services

## 🎯 Objectives

- Understand Services
- Service types
- Service Discovery
- Load Balancing
- Ingress

## 📋 Table of Contents

1. [Introduction to Services](#introduction-to-services)
2. [Service Types](#service-types)
3. [Service Discovery](#service-discovery)
4. [Load Balancing](#load-balancing)
5. [Ingress](#ingress)

---

## Introduction to Services

### What is a Service?

**Service** = Stable access point to Pods

- **Stable IP** : Even if Pods change
- **Load Balancing** : Distributes traffic
- **Service Discovery** : Finds Pods automatically

---

## Service Types

### ClusterIP (default)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 8080
```

### NodePort

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080
```

### LoadBalancer

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - port: 80
```

---

## Service Discovery

### DNS

**Services are accessible by name:**

```python
# In a Pod
import requests
response = requests.get('http://my-service:80')
```

---

## Ingress

### Create an Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-service
            port:
              number: 80
```

---

## 📊 Key Takeaways

1. **Services** : Stable access points
2. **Types** : ClusterIP, NodePort, LoadBalancer
3. **Service Discovery** : By DNS name
4. **Ingress** : HTTP/HTTPS routing

## 🔗 Next Module

Proceed to module [5. ConfigMaps and Secrets](./05-configmaps-secrets/README.md).

