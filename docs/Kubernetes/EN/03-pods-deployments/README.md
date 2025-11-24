# 3. Pods and Deployments

## 🎯 Objectives

- Create Pods
- Manage Deployments
- Scaling and Rolling Updates
- Health Checks

## 📋 Table of Contents

1. [Pods](#pods)
2. [Deployments](#deployments)
3. [Scaling](#scaling)
4. [Rolling Updates](#rolling-updates)
5. [Health Checks](#health-checks)

---

## Pods

### Create a Pod

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

## Deployments

### Create a Deployment

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

## Scaling

### Manual Scaling

```bash
# Scale
kubectl scale deployment nginx-deployment --replicas=5

# See Pods
kubectl get pods
```

### Auto-scaling

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

### Update

```bash
# Update image
kubectl set image deployment/nginx-deployment nginx=nginx:1.22

# See status
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

## 📊 Key Takeaways

1. **Pods** : Basic units
2. **Deployments** : Manage Pods
3. **Scaling** : Manual or automatic
4. **Rolling Updates** : Zero-downtime
5. **Health Checks** : Monitoring

## 🔗 Next Module

Proceed to module [4. Services](./04-services/README.md).

