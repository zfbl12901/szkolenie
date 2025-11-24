# 7. Kubernetes Best Practices

## 🎯 Objectives

- Security
- Performance
- Organization
- Monitoring

## 📋 Table of Contents

1. [Security](#security)
2. [Performance](#performance)
3. [Organization](#organization)
4. [Monitoring](#monitoring)

---

## Security

### Resource Limits

```yaml
resources:
  requests:
    memory: "64Mi"
    cpu: "250m"
  limits:
    memory: "128Mi"
    cpu: "500m"
```

### Security Context

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  fsGroup: 2000
```

---

## Performance

### Resource Requests

```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "100m"
```

### Affinity Rules

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: disktype
          operator: In
          values:
          - ssd
```

---

## Organization

### Labels

```yaml
metadata:
  labels:
    app: my-app
    version: v1
    environment: production
```

### Namespaces

```bash
kubectl create namespace production
kubectl create namespace development
```

---

## Monitoring

### Health Checks

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
```

---

## 📊 Key Takeaways

1. **Security** : Resource limits, security context
2. **Performance** : Resource requests
3. **Organization** : Labels, namespaces
4. **Monitoring** : Health checks

## 🔗 Next Module

Proceed to module [8. Practical Projects](./08-projets/README.md).

