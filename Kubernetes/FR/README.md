# Formation Kubernetes pour Data Analyst

## 📚 Vue d'ensemble

Cette formation vous guide dans l'apprentissage de **Kubernetes** en tant que Data Analyst. Kubernetes est une plateforme open-source pour orchestrer et gérer des conteneurs à grande échelle.

## 🎯 Objectifs pédagogiques

- Comprendre Kubernetes et l'orchestration de conteneurs
- Installer Kubernetes (localement)
- Créer et gérer des Pods et Deployments
- Configurer des Services et Ingress
- Gérer les ConfigMaps et Secrets
- Utiliser les Persistent Volumes
- Créer des projets pratiques pour votre portfolio

## 💰 Tout est gratuit !

Cette formation utilise uniquement :
- ✅ **Minikube / Kind** : Kubernetes local gratuit
- ✅ **kubectl** : CLI Kubernetes gratuite
- ✅ **Documentation officielle** : Guides complets gratuits
- ✅ **Tutoriels en ligne** : Ressources gratuites

**Budget total : 0€**

## 📖 Structure de la formation

### 1. [Prise en main Kubernetes](./01-getting-started/README.md)
   - Installer Kubernetes localement
   - Concepts de base
   - Premiers Pods
   - Commandes essentielles

### 2. [Concepts fondamentaux](./02-concepts/README.md)
   - Architecture Kubernetes
   - Pods, Nodes, Clusters
   - Controllers et ReplicaSets
   - Namespaces

### 3. [Pods et Deployments](./03-pods-deployments/README.md)
   - Créer des Pods
   - Gérer les Deployments
   - Scaling et Rolling Updates
   - Health Checks

### 4. [Services](./04-services/README.md)
   - Types de Services
   - Service Discovery
   - Load Balancing
   - Ingress

### 5. [ConfigMaps et Secrets](./05-configmaps-secrets/README.md)
   - Gérer la configuration
   - Secrets management
   - Variables d'environnement
   - Bonnes pratiques

### 6. [Persistent Volumes](./06-persistent-volumes/README.md)
   - Volumes Kubernetes
   - Persistent Volumes
   - Storage Classes
   - StatefulSets

### 7. [Bonnes pratiques](./07-best-practices/README.md)
   - Sécurité
   - Performance
   - Organisation
   - Monitoring

### 8. [Projets pratiques](./08-projets/README.md)
   - Déployer une application web
   - Pipeline de données avec Kubernetes
   - Stack complète
   - Projets pour portfolio

## 🚀 Démarrage rapide

### Prérequis

- **Docker** : Installé et fonctionnel
- **4 Go RAM** : Minimum recommandé
- **Espace disque** : 20 Go libres

### Installation rapide

**Minikube (recommandé) :**

```bash
# Installer Minikube
# Windows
choco install minikube

# Linux
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# macOS
brew install minikube

# Démarrer Minikube
minikube start

# Vérifier
kubectl get nodes
```

**Kind (alternative) :**

```bash
# Installer Kind
# Windows
choco install kind

# Linux/macOS
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

# Créer un cluster
kind create cluster

# Vérifier
kubectl get nodes
```

### Premier Pod

```bash
# Créer un Pod
kubectl run nginx --image=nginx

# Voir les Pods
kubectl get pods

# Décrire un Pod
kubectl describe pod nginx
```

## 📊 Cas d'usage pour Data Analyst

- **Orchestration** : Gérer plusieurs conteneurs
- **Scaling** : Mettre à l'échelle automatiquement
- **Déploiement** : Déployer facilement
- **Résilience** : Auto-healing
- **Data Pipelines** : Orchestrer des pipelines

## 📚 Ressources gratuites

### Documentation officielle

- **Kubernetes Documentation** : https://kubernetes.io/docs/
- **Kubernetes Playground** : https://www.katacoda.com/courses/kubernetes
- **Minikube** : https://minikube.sigs.k8s.io/

## 🎓 Certifications (optionnel)

### Certified Kubernetes Administrator (CKA)

- **Coût** : ~$395
- **Préparation** : Documentation gratuite
- **Durée** : 2-3 mois
- **Niveau** : Avancé

