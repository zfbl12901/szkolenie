# 8. Kubernetes Practical Projects

## 🎯 Objectives

- Deploy a web application
- Data pipeline with Kubernetes
- Complete stack
- Portfolio projects

## 📋 Table of Contents

1. [Project 1 : Web Application](#project-1--web-application)
2. [Project 2 : Data Pipeline](#project-2--data-pipeline)
3. [Project 3 : Complete Stack](#project-3--complete-stack)

---

## Project 1 : Web Application

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

## Project 2 : Data Pipeline

### Deployment with ConfigMap

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

## Project 3 : Complete Stack

### Application + Database

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

## 📊 Key Takeaways

1. **Deployments** : Manage applications
2. **Services** : Access points
3. **ConfigMaps/Secrets** : Configuration
4. **Volumes** : Persistence
5. **Portfolio** : Demonstrable projects

## 🔗 Resources

- [Kubernetes Examples](https://github.com/kubernetes/examples)
- [Kubernetes Documentation](https://kubernetes.io/docs/)

---

**Congratulations!** You have completed the Kubernetes training.

