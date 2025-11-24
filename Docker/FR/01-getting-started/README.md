# 1. Prise en main Docker

## 🎯 Objectifs

- Comprendre Docker et la conteneurisation
- Installer Docker
- Comprendre les concepts de base
- Exécuter votre premier conteneur

## 📋 Table des matières

1. [Introduction à Docker](#introduction-à-docker)
2. [Installation](#installation)
3. [Concepts de base](#concepts-de-base)
4. [Premiers conteneurs](#premiers-conteneurs)
5. [Commandes essentielles](#commandes-essentielles)

---

## Introduction à Docker

### Qu'est-ce que Docker ?

**Docker** = Plateforme de conteneurisation

- **Conteneurs** : Environnements isolés et légers
- **Portable** : Fonctionne partout (Windows, Linux, macOS)
- **Efficace** : Utilise moins de ressources que les VMs
- **Rapide** : Démarrage en secondes

### Pourquoi Docker pour Data Analyst ?

- **Reproductibilité** : Même environnement partout
- **Isolation** : Séparer les dépendances Python/R
- **Simplicité** : Facile à partager et déployer
- **Performance** : Plus rapide que les VMs

### Docker vs Virtual Machines

**Docker (Conteneurs) :**
- Plus léger
- Démarrage rapide
- Partage le noyau OS
- Moins de ressources

**Virtual Machines :**
- Plus lourd
- Démarrage plus lent
- OS complet
- Plus de ressources

---

## Installation

### Windows

**Étape 1 : Télécharger Docker Desktop**

1. Aller sur : https://www.docker.com/products/docker-desktop
2. Télécharger Docker Desktop pour Windows
3. Installer le fichier `.exe`
4. Redémarrer l'ordinateur si demandé

**Étape 2 : Lancer Docker Desktop**

1. Ouvrir Docker Desktop
2. Attendre que Docker démarre (icône dans la barre des tâches)
3. Vérifier : `docker --version`

**Prérequis Windows :**
- Windows 10 64-bit : Pro, Enterprise, ou Education
- WSL 2 activé
- Virtualisation activée dans le BIOS

### Linux

**Ubuntu/Debian :**

```bash
# Mettre à jour les paquets
sudo apt update

# Installer les dépendances
sudo apt install apt-transport-https ca-certificates curl gnupg lsb-release

# Ajouter la clé GPG Docker
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Ajouter le repository Docker
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Installer Docker
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io

# Démarrer Docker
sudo systemctl start docker
sudo systemctl enable docker

# Vérifier
docker --version
```

**CentOS/RHEL :**

```bash
# Installer les dépendances
sudo yum install -y yum-utils

# Ajouter le repository Docker
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# Installer Docker
sudo yum install docker-ce docker-ce-cli containerd.io

# Démarrer Docker
sudo systemctl start docker
sudo systemctl enable docker

# Vérifier
docker --version
```

### macOS

**Étape 1 : Télécharger Docker Desktop**

1. Aller sur : https://www.docker.com/products/docker-desktop
2. Télécharger Docker Desktop pour Mac
3. Installer le fichier `.dmg`
4. Glisser Docker dans Applications

**Étape 2 : Lancer Docker Desktop**

1. Ouvrir Docker depuis Applications
2. Attendre que Docker démarre
3. Vérifier : `docker --version`

---

## Concepts de base

### Images

**Image** = Modèle en lecture seule pour créer des conteneurs

- **Template** : Contient l'OS, les applications, les dépendances
- **Immuable** : Ne change pas une fois créée
- **Légère** : Partage les couches communes

### Conteneurs

**Conteneur** = Instance exécutable d'une image

- **Isolé** : Environnement séparé
- **Éphémère** : Peut être créé/détruit facilement
- **Portable** : Fonctionne partout où Docker est installé

### Dockerfile

**Dockerfile** = Instructions pour construire une image

- **Définit** : L'environnement et les applications
- **Automatise** : La création d'images
- **Versionne** : Peut être versionné avec Git

### Docker Hub

**Docker Hub** = Registre public d'images Docker

- **Images publiques** : Python, PostgreSQL, Redis, etc.
- **Gratuit** : Pour usage public
- **Partage** : Partagez vos images

---

## Premiers conteneurs

### Hello World

```bash
# Exécuter le conteneur Hello World
docker run hello-world
```

**Ce qui se passe :**
1. Docker télécharge l'image `hello-world` (si pas présente)
2. Crée un conteneur
3. Exécute le conteneur
4. Affiche le message
5. Arrête le conteneur

### Conteneur interactif

```bash
# Exécuter un conteneur Ubuntu interactif
docker run -it ubuntu bash

# Dans le conteneur
ls
pwd
exit
```

**Options :**
- `-i` : Mode interactif (stdin)
- `-t` : Allouer un terminal
- `ubuntu` : Image à utiliser
- `bash` : Commande à exécuter

### Conteneur en arrière-plan

```bash
# Exécuter un conteneur en arrière-plan
docker run -d --name my-container nginx

# Voir les conteneurs en cours
docker ps

# Voir les logs
docker logs my-container

# Arrêter le conteneur
docker stop my-container
```

---

## Commandes essentielles

### Gestion des conteneurs

```bash
# Lister les conteneurs en cours
docker ps

# Lister tous les conteneurs
docker ps -a

# Créer un conteneur
docker create --name my-container ubuntu

# Démarrer un conteneur
docker start my-container

# Arrêter un conteneur
docker stop my-container

# Redémarrer un conteneur
docker restart my-container

# Supprimer un conteneur
docker rm my-container

# Supprimer un conteneur en cours (force)
docker rm -f my-container
```

### Gestion des images

```bash
# Lister les images
docker images

# Télécharger une image
docker pull ubuntu

# Supprimer une image
docker rmi ubuntu

# Chercher des images sur Docker Hub
docker search python
```

### Exécution de commandes

```bash
# Exécuter une commande dans un conteneur
docker exec my-container ls

# Exécuter une commande interactive
docker exec -it my-container bash

# Voir les logs
docker logs my-container

# Suivre les logs en temps réel
docker logs -f my-container
```

### Informations

```bash
# Informations système Docker
docker info

# Version Docker
docker --version

# Statistiques des conteneurs
docker stats

# Inspecter un conteneur
docker inspect my-container
```

---

## Exemples pratiques

### Exemple 1 : Conteneur Python

```bash
# Exécuter Python dans un conteneur
docker run -it python:3.11 python

# Dans Python
print("Hello from Docker!")
exit()
```

### Exemple 2 : Conteneur avec volume

```bash
# Créer un fichier local
echo "print('Hello from file')" > script.py

# Exécuter Python avec volume
docker run -v $(pwd):/app -w /app python:3.11 python script.py
```

### Exemple 3 : Conteneur avec port

```bash
# Exécuter un serveur web
docker run -d -p 8080:80 --name web-server nginx

# Accéder au serveur
# Ouvrir : http://localhost:8080

# Arrêter
docker stop web-server
docker rm web-server
```

---

## Dépannage

### Problème : Docker ne démarre pas

**Solutions :**
1. Vérifier que la virtualisation est activée (BIOS)
2. Vérifier WSL 2 (Windows)
3. Redémarrer Docker Desktop
4. Vérifier les logs : Docker Desktop → Troubleshoot

### Problème : Permission denied (Linux)

**Solutions :**
```bash
# Ajouter l'utilisateur au groupe docker
sudo usermod -aG docker $USER

# Se déconnecter et reconnecter
# Ou
newgrp docker
```

### Problème : Conteneur ne démarre pas

**Solutions :**
1. Vérifier les logs : `docker logs container-name`
2. Vérifier les ressources : `docker stats`
3. Vérifier la configuration : `docker inspect container-name`

---

## 📊 Points clés à retenir

1. **Docker = Conteneurisation** pour isoler les applications
2. **Images** sont les modèles, **Conteneurs** sont les instances
3. **Docker Hub** pour trouver des images
4. **Commandes de base** : run, ps, stop, rm
5. **Portable** : Fonctionne partout

## 🔗 Prochain module

Passer au module [2. Conteneurs](./02-containers/README.md) pour approfondir la gestion des conteneurs.

