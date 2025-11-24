# 2. Concepts fondamentaux Kubernetes

## 🎯 Objectifs

- Comprendre l'architecture Kubernetes
- Maîtriser Pods, Nodes, Clusters
- Comprendre Controllers et ReplicaSets
- Utiliser les Namespaces

## 📋 Table des matières

1. [Architecture](#architecture)
2. [Pods](#pods)
3. [Nodes](#nodes)
4. [Controllers](#controllers)
5. [Namespaces](#namespaces)

---

## Architecture

### Composants principaux

**Master Node :**
- **API Server** : Point d'entrée
- **etcd** : Base de données
- **Scheduler** : Planifie les Pods
- **Controller Manager** : Gère les controllers

**Worker Node :**
- **kubelet** : Agent sur chaque node
- **kube-proxy** : Réseau
- **Container Runtime** : Docker/containerd

---

## Pods

### Qu'est-ce qu'un Pod ?

**Pod** = Plus petite unité déployable

- **Un ou plusieurs conteneurs** : Partagent réseau/stockage
- **Éphémère** : Peut être recréé
- **IP unique** : Chaque Pod a une IP

### Exemple de Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: app
    image: nginx
    ports:
    - containerPort: 80
```

---

## Nodes

### Types de Nodes

**Master Node :**
- Gère le cluster
- Planifie les Pods
- Gère l'état

**Worker Node :**
- Exécute les Pods
- Fournit les ressources

---

## Controllers

### Types de Controllers

**ReplicaSet :**
- Maintient un nombre de Pods
- Auto-healing

**Deployment :**
- Gère les ReplicaSets
- Rolling updates

**StatefulSet :**
- Pour applications stateful
- Identité stable

---

## Namespaces

### Utiliser les Namespaces

```bash
# Lister les namespaces
kubectl get namespaces

# Créer un namespace
kubectl create namespace my-namespace

# Utiliser un namespace
kubectl get pods -n my-namespace
```

---

## 📊 Points clés à retenir

1. **Architecture** : Master et Worker Nodes
2. **Pods** : Plus petites unités
3. **Controllers** : Gèrent les Pods
4. **Namespaces** : Isolation logique

## 🔗 Prochain module

Passer au module [3. Pods et Deployments](./03-pods-deployments/README.md).

