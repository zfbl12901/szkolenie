# 5. ConfigMaps i Secrety

## 🎯 Cele

- Zarządzać konfiguracją z ConfigMapami
- Zarządzać secretami
- Zmienne środowiskowe
- Dobre praktyki

## 📋 Spis treści

1. [ConfigMaps](#configmaps)
2. [Secrety](#secrety)
3. [Użycie](#użycie)
4. [Dobre praktyki](#dobre-praktyki)

---

## ConfigMaps

### Utworzyć ConfigMap

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

### Używać w Podzie

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

## Secrety

### Utworzyć Secret

```bash
# Utworzyć z linii poleceń
kubectl create secret generic my-secret \
  --from-literal=username=admin \
  --from-literal=password=secret123
```

### Używać w Podzie

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

## 📊 Kluczowe punkty do zapamiętania

1. **ConfigMaps** : Konfiguracja niewrażliwa
2. **Secrety** : Dane wrażliwe
3. **Zmienne środowiskowe** : Wstrzykiwanie do Podów
4. **Bezpieczeństwo** : Nie committować secretów

## 🔗 Następny moduł

Przejdź do modułu [6. Persistent Volumes](./06-persistent-volumes/README.md).

