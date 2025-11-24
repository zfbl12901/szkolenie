# 8. Projekty praktyczne Kubernetes

## 🎯 Cele

- Wdrożyć aplikację web
- Pipeline danych z Kubernetes
- Kompletny stack
- Projekty do portfolio

## 📋 Spis treści

1. [Projekt 1 : Aplikacja web](#projekt-1--aplikacja-web)
2. [Projekt 2 : Pipeline danych](#projekt-2--pipeline-danych)
3. [Projekt 3 : Kompletny stack](#projekt-3--kompletny-stack)

---

## Projekt 1 : Aplikacja web

### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: app
        image: nginx:latest
        ports:
        - containerPort: 80
```

### Serwis

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
  type: LoadBalancer
```

---

## Projekt 2 : Pipeline danych

### Deployment z ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: pipeline-config
data:
  config.yaml: |
    input_path: /data/input
    output_path: /data/output

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: data-pipeline
spec:
  replicas: 2
  selector:
    matchLabels:
      app: pipeline
  template:
    metadata:
      labels:
        app: pipeline
    spec:
      containers:
      - name: pipeline
        image: python:3.11
        command: ["python", "pipeline.py"]
        volumeMounts:
        - name: config
          mountPath: /config
        - name: data
          mountPath: /data
      volumes:
      - name: config
        configMap:
          name: pipeline-config
      - name: data
        persistentVolumeClaim:
          claimName: data-pvc
```

---

## Projekt 3 : Kompletny stack

### Aplikacja + Baza danych

```yaml
# Deployment app
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: app
        image: myapp:latest
        env:
        - name: DB_HOST
          value: "db-service"
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password

---
# Serwis app
apiVersion: v1
kind: Service
metadata:
  name: app-service
spec:
  selector:
    app: myapp
  ports:
  - port: 80

---
# Deployment DB
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: db
spec:
  serviceName: db-service
  replicas: 1
  selector:
    matchLabels:
      app: db
  template:
    metadata:
      labels:
        app: db
    spec:
      containers:
      - name: postgres
        image: postgres:15
        volumeMounts:
        - name: db-storage
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: db-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **Deploymenty** : Zarządzają aplikacjami
2. **Serwisy** : Punkty dostępu
3. **ConfigMaps/Secrety** : Konfiguracja
4. **Wolumeny** : Trwałość
5. **Portfolio** : Projekty demonstrowalne

## 🔗 Zasoby

- [Przykłady Kubernetes](https://github.com/kubernetes/examples)
- [Dokumentacja Kubernetes](https://kubernetes.io/docs/)

---

**Gratulacje !** Ukończyłeś szkolenie Kubernetes.

