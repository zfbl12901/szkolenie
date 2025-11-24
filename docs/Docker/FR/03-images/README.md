# 3. Images Docker

## 🎯 Objectifs

- Comprendre les images Docker
- Télécharger des images
- Créer des images personnalisées
- Gérer les images
- Optimiser les images

## 📋 Table des matières

1. [Introduction aux images](#introduction-aux-images)
2. [Télécharger des images](#télécharger-des-images)
3. [Créer des images](#créer-des-images)
4. [Gérer les images](#gérer-les-images)
5. [Optimisation](#optimisation)

---

## Introduction aux images

### Qu'est-ce qu'une image ?

**Image** = Modèle en lecture seule

- **Template** : Pour créer des conteneurs
- **Layering** : Composée de couches
- **Immuable** : Ne change pas
- **Partagée** : Plusieurs conteneurs peuvent utiliser la même image

### Structure d'une image

```
Image
├── Couche 1 : OS de base (Ubuntu)
├── Couche 2 : Outils système
├── Couche 3 : Python
└── Couche 4 : Votre application
```

---

## Télécharger des images

### Docker Hub

**Docker Hub** = Registre public d'images

- **Images officielles** : python, postgres, nginx, etc.
- **Images communautaires** : Créées par la communauté
- **Gratuit** : Pour usage public

### Télécharger une image

```bash
# Télécharger une image
docker pull python:3.11

# Télécharger la dernière version
docker pull python:latest

# Télécharger une version spécifique
docker pull python:3.11-slim

# Chercher des images
docker search python
```

### Images populaires pour Data Analyst

```bash
# Python
docker pull python:3.11

# Jupyter Notebook
docker pull jupyter/scipy-notebook

# PostgreSQL
docker pull postgres:15

# MySQL
docker pull mysql:8.0

# Redis
docker pull redis:7
```

---

## Créer des images

### Avec Dockerfile

**Créer un Dockerfile :**

```dockerfile
# Dockerfile
FROM python:3.11

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

CMD ["python", "app.py"]
```

**Construire l'image :**

```bash
# Construire une image
docker build -t my-app:latest .

# Avec tag spécifique
docker build -t my-app:v1.0 .

# Depuis un Dockerfile spécifique
docker build -f Dockerfile.prod -t my-app:prod .
```

### Commit depuis un conteneur

```bash
# Créer un conteneur
docker run -it ubuntu bash

# Faire des modifications dans le conteneur
apt update
apt install python3

# Créer une image depuis le conteneur
docker commit container-id my-image:tag
```

---

## Gérer les images

### Lister les images

```bash
# Lister toutes les images
docker images

# Filtrer par nom
docker images python

# Afficher seulement les IDs
docker images -q
```

### Supprimer des images

```bash
# Supprimer une image
docker rmi my-image:tag

# Supprimer par ID
docker rmi image-id

# Supprimer toutes les images non utilisées
docker image prune

# Supprimer toutes les images
docker rmi $(docker images -q)
```

### Taguer des images

```bash
# Créer un tag
docker tag my-image:latest my-image:v1.0

# Taguer pour Docker Hub
docker tag my-image:latest username/my-image:latest
```

### Inspecter une image

```bash
# Informations complètes
docker inspect my-image

# Historique des couches
docker history my-image

# Taille de l'image
docker images my-image
```

---

## Optimisation

### Images légères

**Utiliser des images slim :**

```dockerfile
# Au lieu de
FROM python:3.11

# Utiliser
FROM python:3.11-slim
```

**Multi-stage builds :**

```dockerfile
# Stage 1 : Build
FROM python:3.11 as builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user -r requirements.txt

# Stage 2 : Runtime
FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY . .
CMD ["python", "app.py"]
```

### Réduire la taille

**Bonnes pratiques :**
1. Utiliser `.dockerignore`
2. Combiner les RUN
3. Utiliser des images de base légères
4. Nettoyer les caches

---

## 📊 Points clés à retenir

1. **Images** sont les modèles pour conteneurs
2. **Docker Hub** pour trouver des images
3. **Dockerfile** pour créer des images
4. **Layering** permet le partage
5. **Optimisation** réduit la taille

## 🔗 Prochain module

Passer au module [4. Dockerfile](./04-dockerfile/README.md) pour apprendre à écrire des Dockerfiles.

