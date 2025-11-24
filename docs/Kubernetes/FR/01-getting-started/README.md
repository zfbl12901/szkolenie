# 1. Prise en main Kubernetes

## 🎯 Objectifs

- Comprendre Kubernetes
- Installer Kubernetes localement
- Comprendre les concepts de base
- Créer votre premier Pod

## 📋 Table des matières

1. [Introduction à Kubernetes](#introduction-à-kubernetes)
2. [Installation](#installation)
3. [Concepts de base](#concepts-de-base)
4. [Premiers Pods](#premiers-pods)
5. [Commandes essentielles](#commandes-essentielles)

---

## Introduction à Kubernetes

### Qu'est-ce que Kubernetes ?

**Kubernetes (K8s)** = Plateforme d'orchestration de conteneurs

- **Orchestration** : Gère plusieurs conteneurs
- **Scaling** : Mise à l'échelle automatique
- **Auto-healing** : Redémarre les conteneurs défaillants
- **Load Balancing** : Répartition de charge
- **Rolling Updates** : Mises à jour sans interruption

### Pourquoi Kubernetes pour Data Analyst ?

- **Orchestration** : Gérer plusieurs services
- **Scaling** : Adapter aux besoins
- **Résilience** : Auto-healing
- **Déploiement** : Déployer facilement

---

## Installation

### Minikube (recommandé)

**Windows :**
```bash
# Avec Chocolatey
choco install minikube

# Ou télécharger
# https://minikube.sigs.k8s.io/docs/start/

# Démarrer
minikube start

# Vérifier
kubectl get nodes
```

**Linux :**
```bash
# Installer Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Démarrer
minikube start

# Vérifier
kubectl get nodes
```

**macOS :**
```bash
# Avec Homebrew
brew install minikube

# Démarrer
minikube start

# Vérifier
kubectl get nodes
```

### Kind (alternative)

```bash
# Installer Kind
# Windows
choco install kind

# Linux/macOS
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

# Créer un cluster
kind create cluster --name my-cluster

# Vérifier
kubectl get nodes
```

### Installer kubectl

**kubectl** = CLI pour Kubernetes

**Windows :**
```bash
# Avec Chocolatey
choco install kubernetes-cli

# Ou télécharger
# https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/
```

**Linux :**
```bash
# Installer kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

**macOS :**
```bash
# Avec Homebrew
brew install kubectl
```

---

## Concepts de base

### Cluster

**Cluster** = Ensemble de machines (nodes)

- **Master Node** : Gère le cluster
- **Worker Nodes** : Exécutent les conteneurs

### Pod

**Pod** = Plus petit déploiement dans Kubernetes

- **Conteneurs** : Un ou plusieurs conteneurs
- **Ressources partagées** : Réseau et stockage
- **Éphémère** : Peut être créé/détruit

### Node

**Node** = Machine dans le cluster

- **Worker Node** : Exécute les Pods
- **Master Node** : Gère le cluster

---

## Premiers Pods

### Créer un Pod simple

```bash
# Créer un Pod
kubectl run nginx --image=nginx

# Voir les Pods
kubectl get pods

# Décrire un Pod
kubectl describe pod nginx

# Logs d'un Pod
kubectl logs nginx
```

### Pod avec YAML

**nginx-pod.yaml :**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

**Créer le Pod :**
```bash
kubectl apply -f nginx-pod.yaml

# Voir le Pod
kubectl get pods

# Supprimer le Pod
kubectl delete -f nginx-pod.yaml
```

---

## Commandes essentielles

### Gestion des Pods

```bash
# Lister les Pods
kubectl get pods

# Décrire un Pod
kubectl describe pod pod-name

# Logs d'un Pod
kubectl logs pod-name

# Exécuter une commande dans un Pod
kubectl exec -it pod-name -- bash

# Supprimer un Pod
kubectl delete pod pod-name
```

### Informations

```bash
# Voir les nodes
kubectl get nodes

# Voir les namespaces
kubectl get namespaces

# Informations du cluster
kubectl cluster-info

# Version
kubectl version
```

---

## 📊 Points clés à retenir

1. **Kubernetes** orchestre les conteneurs
2. **Pods** sont les plus petites unités
3. **kubectl** est la CLI principale
4. **Minikube/Kind** pour Kubernetes local
5. **YAML** pour définir les ressources

## 🔗 Prochain module

Passer au module [2. Concepts fondamentaux](./02-concepts/README.md) pour approfondir.

