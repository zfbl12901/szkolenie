# 3. Pody i Deploymenty

## 🎯 Cele

- Tworzyć Pody
- Zarządzać Deploymentami
- Skalowanie i Rolling Updates
- Health Checks

## 📋 Spis treści

1. [Pody](#pody)
2. [Deploymenty](#deploymenty)
3. [Skalowanie](#skalowanie)
4. [Rolling Updates](#rolling-updates)
5. [Health Checks](#health-checks)

---

## Pody

### Utworzyć Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

```bash
kubectl apply -f pod.yaml
```

---

## Deploymenty

### Utworzyć Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
```

```bash
kubectl apply -f deployment.yaml
kubectl get deployments
```

---

## Skalowanie

### Skalowanie ręczne

```bash
# Skalować
kubectl scale deployment nginx-deployment --replicas=5

# Zobaczyć Pody
kubectl get pods
```

### Auto-skaling

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

---

## Rolling Updates

### Aktualizować

```bash
# Aktualizować obraz
kubectl set image deployment/nginx-deployment nginx=nginx:1.22

# Zobaczyć status
kubectl rollout status deployment/nginx-deployment

# Rollback
kubectl rollout undo deployment/nginx-deployment
```

---

## Health Checks

### Liveness Probe

```yaml
containers:
- name: nginx
  image: nginx
  livenessProbe:
    httpGet:
      path: /
      port: 80
    initialDelaySeconds: 30
    periodSeconds: 10
```

### Readiness Probe

```yaml
readinessProbe:
  httpGet:
    path: /health
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 5
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **Pody** : Podstawowe jednostki
2. **Deploymenty** : Zarządzają Podami
3. **Skalowanie** : Ręczne lub automatyczne
4. **Rolling Updates** : Bez przestojów
5. **Health Checks** : Monitorowanie

## 🔗 Następny moduł

Przejdź do modułu [4. Serwisy](./04-services/README.md).

