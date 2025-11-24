# 7. Bonnes pratiques Kubernetes

## 🎯 Objectifs

- Sécurité
- Performance
- Organisation
- Monitoring

## 📋 Table des matières

1. [Sécurité](#sécurité)
2. [Performance](#performance)
3. [Organisation](#organisation)
4. [Monitoring](#monitoring)

---

## Sécurité

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

## Organisation

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

## 📊 Points clés à retenir

1. **Sécurité** : Resource limits, security context
2. **Performance** : Resource requests
3. **Organisation** : Labels, namespaces
4. **Monitoring** : Health checks

## 🔗 Prochain module

Passer au module [8. Projets pratiques](./08-projets/README.md).

