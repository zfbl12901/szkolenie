# 4. Serwisy Kubernetes

## 🎯 Cele

- Zrozumieć Serwisy
- Typy serwisów
- Service Discovery
- Load Balancing
- Ingress

## 📋 Spis treści

1. [Wprowadzenie do Serwisów](#wprowadzenie-do-serwisów)
2. [Typy serwisów](#typy-serwisów)
3. [Service Discovery](#service-discovery)
4. [Load Balancing](#load-balancing)
5. [Ingress](#ingress)

---

## Wprowadzenie do Serwisów

### Czym jest Serwis?

**Serwis** = Stabilny punkt dostępu do Podów

- **Stabilne IP** : Nawet jeśli Pody się zmieniają
- **Load Balancing** : Rozkłada ruch
- **Service Discovery** : Znajduje Pody automatycznie

---

## Typy serwisów

### ClusterIP (domyślny)

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

**Serwisy są dostępne po nazwie:**

```python
# W Podzie
import requests
response = requests.get('http://my-service:80')
```

---

## Ingress

### Utworzyć Ingress

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

## 📊 Kluczowe punkty do zapamiętania

1. **Serwisy** : Stabilne punkty dostępu
2. **Typy** : ClusterIP, NodePort, LoadBalancer
3. **Service Discovery** : Po nazwie DNS
4. **Ingress** : Routing HTTP/HTTPS

## 🔗 Następny moduł

Przejdź do modułu [5. ConfigMaps i Secrety](./05-configmaps-secrets/README.md).

