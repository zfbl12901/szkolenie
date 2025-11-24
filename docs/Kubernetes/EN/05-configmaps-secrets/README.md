# 5. ConfigMaps and Secrets

## 🎯 Objectives

- Manage configuration with ConfigMaps
- Manage secrets
- Environment variables
- Best practices

## 📋 Table of Contents

1. [ConfigMaps](#configmaps)
2. [Secrets](#secrets)
3. [Usage](#usage)
4. [Best Practices](#best-practices)

---

## ConfigMaps

### Create a ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  config.yaml: |
    database:
      host: localhost
      port: 5432
  app.properties: |
    debug=true
    log_level=info
```

```bash
kubectl apply -f configmap.yaml
```

### Use in a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: app
    image: nginx
    envFrom:
    - configMapRef:
        name: my-config
```

---

## Secrets

### Create a Secret

```bash
# Create from command line
kubectl create secret generic my-secret \
  --from-literal=username=admin \
  --from-literal=password=secret123
```

### Use in a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: app
    image: nginx
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: password
```

---

## 📊 Key Takeaways

1. **ConfigMaps** : Non-sensitive configuration
2. **Secrets** : Sensitive data
3. **Environment variables** : Injection into Pods
4. **Security** : Don't commit secrets

## 🔗 Next Module

Proceed to module [6. Persistent Volumes](./06-persistent-volumes/README.md).

