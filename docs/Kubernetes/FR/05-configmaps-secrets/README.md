# 5. ConfigMaps et Secrets

## 🎯 Objectifs

- Gérer la configuration avec ConfigMaps
- Gérer les secrets
- Variables d'environnement
- Bonnes pratiques

## 📋 Table des matières

1. [ConfigMaps](#configmaps)
2. [Secrets](#secrets)
3. [Utilisation](#utilisation)
4. [Bonnes pratiques](#bonnes-pratiques)

---

## ConfigMaps

### Créer un ConfigMap

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

### Utiliser dans un Pod

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

### Créer un Secret

```bash
# Créer depuis la ligne de commande
kubectl create secret generic my-secret \
  --from-literal=username=admin \
  --from-literal=password=secret123
```

### Utiliser dans un Pod

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

## 📊 Points clés à retenir

1. **ConfigMaps** : Configuration non sensible
2. **Secrets** : Données sensibles
3. **Variables d'environnement** : Injection dans Pods
4. **Sécurité** : Ne pas commiter les secrets

## 🔗 Prochain module

Passer au module [6. Persistent Volumes](./06-persistent-volumes/README.md).

