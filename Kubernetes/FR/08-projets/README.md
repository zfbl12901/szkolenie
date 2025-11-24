# 8. Projets pratiques Kubernetes

## 🎯 Objectifs

- Déployer une application web
- Pipeline de données avec Kubernetes
- Stack complète
- Projets pour portfolio

## 📋 Table des matières

1. [Projet 1 : Application web](#projet-1--application-web)
2. [Projet 2 : Pipeline de données](#projet-2--pipeline-de-données)
3. [Projet 3 : Stack complète](#projet-3--stack-complète)

---

## Projet 1 : Application web

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

### Service

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

## Projet 2 : Pipeline de données

### Deployment avec ConfigMap

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

## Projet 3 : Stack complète

### Application + Base de données

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
# Service app
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

## 📊 Points clés à retenir

1. **Deployments** : Gèrent les applications
2. **Services** : Points d'accès
3. **ConfigMaps/Secrets** : Configuration
4. **Volumes** : Persistance
5. **Portfolio** : Projets démontrables

## 🔗 Ressources

- [Kubernetes Examples](https://github.com/kubernetes/examples)
- [Kubernetes Documentation](https://kubernetes.io/docs/)

---

**Félicitations !** Vous avez terminé la formation Kubernetes.

