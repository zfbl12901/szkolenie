# 6. Persistent Volumes

## 🎯 Objectifs

- Comprendre les Volumes Kubernetes
- Persistent Volumes et Claims
- Storage Classes
- StatefulSets

## 📋 Table des matières

1. [Volumes](#volumes)
2. [Persistent Volumes](#persistent-volumes)
3. [Storage Classes](#storage-classes)
4. [StatefulSets](#statefulsets)

---

## Volumes

### Types de Volumes

- **emptyDir** : Temporaire
- **hostPath** : Répertoire hôte
- **PersistentVolume** : Stockage persistant

---

## Persistent Volumes

### Créer un PersistentVolume

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

## StatefulSets

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

## 📊 Points clés à retenir

1. **Volumes** : Stockage temporaire
2. **Persistent Volumes** : Stockage persistant
3. **Storage Classes** : Provisionnement dynamique
4. **StatefulSets** : Applications stateful

## 🔗 Prochain module

Passer au module [7. Bonnes pratiques](./07-best-practices/README.md).

