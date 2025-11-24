# 4. Services Kubernetes

## 🎯 Objectifs

- Comprendre les Services
- Types de Services
- Service Discovery
- Load Balancing
- Ingress

## 📋 Table des matières

1. [Introduction aux Services](#introduction-aux-services)
2. [Types de Services](#types-de-services)
3. [Service Discovery](#service-discovery)
4. [Load Balancing](#load-balancing)
5. [Ingress](#ingress)

---

## Introduction aux Services

### Qu'est-ce qu'un Service ?

**Service** = Point d'accès stable aux Pods

- **IP stable** : Même si les Pods changent
- **Load Balancing** : Répartit le trafic
- **Service Discovery** : Trouve les Pods automatiquement

---

## Types de Services

### ClusterIP (par défaut)

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

**Les Services sont accessibles par nom :**

```python
# Dans un Pod
import requests
response = requests.get('http://my-service:80')
```

---

## Ingress

### Créer un Ingress

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

## 📊 Points clés à retenir

1. **Services** : Points d'accès stables
2. **Types** : ClusterIP, NodePort, LoadBalancer
3. **Service Discovery** : Par nom DNS
4. **Ingress** : Routage HTTP/HTTPS

## 🔗 Prochain module

Passer au module [5. ConfigMaps et Secrets](./05-configmaps-secrets/README.md).

