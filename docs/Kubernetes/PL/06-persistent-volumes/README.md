# 6. Persistent Volumes

## 🎯 Cele

- Zrozumieć Wolumeny Kubernetes
- Persistent Volumes i Claims
- Storage Classes
- StatefulSety

## 📋 Spis treści

1. [Wolumeny](#wolumeny)
2. [Persistent Volumes](#persistent-volumes)
3. [Storage Classes](#storage-classes)
4. [StatefulSety](#statefulsety)

---

## Wolumeny

### Typy wolumenów

- **emptyDir** : Tymczasowy
- **hostPath** : Katalog hosta
- **PersistentVolume** : Magazyn trwały

---

## Persistent Volumes

### Utworzyć PersistentVolume

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data
```

### PersistentVolumeClaim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
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
    volumeMounts:
    - name: storage
      mountPath: /data
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: my-pvc
```

---

## Storage Classes

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
```

---

## StatefulSety

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
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
        image: nginx
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **Wolumeny** : Magazyn tymczasowy
2. **Persistent Volumes** : Magazyn trwały
3. **Storage Classes** : Provisioning dynamiczny
4. **StatefulSety** : Aplikacje stateful

## 🔗 Następny moduł

Przejdź do modułu [7. Dobre praktyki](./07-best-practices/README.md).

