# 7. Dobre praktyki Kubernetes

## 🎯 Cele

- Bezpieczeństwo
- Wydajność
- Organizacja
- Monitorowanie

## 📋 Spis treści

1. [Bezpieczeństwo](#bezpieczeństwo)
2. [Wydajność](#wydajność)
3. [Organizacja](#organizacja)
4. [Monitorowanie](#monitorowanie)

---

## Bezpieczeństwo

### Limity zasobów

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

## Wydajność

### Żądania zasobów

```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "100m"
```

### Reguły Affinity

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

## Organizacja

### Etykiety

```yaml
metadata:
  labels:
    app: my-app
    version: v1
    environment: production
```

### Namespace'y

```bash
kubectl create namespace production
kubectl create namespace development
```

---

## Monitorowanie

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

## 📊 Kluczowe punkty do zapamiętania

1. **Bezpieczeństwo** : Limity zasobów, security context
2. **Wydajność** : Żądania zasobów
3. **Organizacja** : Etykiety, namespace'y
4. **Monitorowanie** : Health checks

## 🔗 Następny moduł

Przejdź do modułu [8. Projekty praktyczne](./08-projets/README.md).

