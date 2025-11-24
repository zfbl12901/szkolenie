# 4. Dockerfile

## 🎯 Objectifs

- Écrire un Dockerfile
- Comprendre les instructions
- Optimiser les Dockerfiles
- Utiliser les multi-stage builds
- Bonnes pratiques

## 📋 Table des matières

1. [Introduction au Dockerfile](#introduction-au-dockerfile)
2. [Instructions de base](#instructions-de-base)
3. [Dockerfile complet](#dockerfile-complet)
4. [Optimisation](#optimisation)
5. [Multi-stage builds](#multi-stage-builds)

---

## Introduction au Dockerfile

### Qu'est-ce qu'un Dockerfile ?

**Dockerfile** = Instructions pour construire une image

- **Texte** : Fichier texte simple
- **Instructions** : Chaque ligne est une instruction
- **Automatisation** : Automatise la création d'images
- **Versionné** : Peut être versionné avec Git

### Structure de base

```dockerfile
# Commentaire
FROM base-image
RUN command
COPY source destination
CMD ["executable", "param"]
```

---

## Instructions de base

### FROM

**Définit l'image de base :**

```dockerfile
FROM python:3.11
FROM ubuntu:22.04
FROM alpine:latest
```

### WORKDIR

**Définit le répertoire de travail :**

```dockerfile
WORKDIR /app
WORKDIR /usr/src/app
```

### COPY / ADD

**Copier des fichiers :**

```dockerfile
# COPY (recommandé)
COPY requirements.txt .
COPY . .

# ADD (avec extraction automatique)
ADD archive.tar.gz /app
```

### RUN

**Exécuter des commandes :**

```dockerfile
RUN apt update
RUN pip install -r requirements.txt

# Combiner pour réduire les couches
RUN apt update && \
    apt install -y python3 && \
    apt clean
```

### CMD / ENTRYPOINT

**Commande par défaut :**

```dockerfile
# CMD (peut être override)
CMD ["python", "app.py"]

# ENTRYPOINT (ne peut pas être override)
ENTRYPOINT ["python"]
CMD ["app.py"]
```

### ENV

**Variables d'environnement :**

```dockerfile
ENV PYTHONUNBUFFERED=1
ENV APP_ENV=production
```

### EXPOSE

**Exposer des ports :**

```dockerfile
EXPOSE 8080
EXPOSE 3306
```

---

## Dockerfile complet

### Exemple 1 : Application Python

```dockerfile
# Image de base
FROM python:3.11-slim

# Répertoire de travail
WORKDIR /app

# Variables d'environnement
ENV PYTHONUNBUFFERED=1

# Copier requirements
COPY requirements.txt .

# Installer les dépendances
RUN pip install --no-cache-dir -r requirements.txt

# Copier le code
COPY . .

# Exposer le port
EXPOSE 8000

# Commande par défaut
CMD ["python", "app.py"]
```

### Exemple 2 : Application avec données

```dockerfile
FROM python:3.11

WORKDIR /app

# Installer les dépendances système
RUN apt update && \
    apt install -y postgresql-client && \
    apt clean

# Installer les dépendances Python
COPY requirements.txt .
RUN pip install -r requirements.txt

# Copier l'application
COPY . .

# Créer un volume pour les données
VOLUME ["/app/data"]

# Exposer le port
EXPOSE 8080

# Commande
CMD ["python", "main.py"]
```

---

## Optimisation

### .dockerignore

**Créer un fichier `.dockerignore` :**

```
__pycache__
*.pyc
.git
.env
node_modules
*.log
.DS_Store
```

### Réduire les couches

**Mauvais :**
```dockerfile
RUN apt update
RUN apt install -y python3
RUN apt install -y pip
RUN apt clean
```

**Bon :**
```dockerfile
RUN apt update && \
    apt install -y python3 pip && \
    apt clean
```

### Ordre des instructions

**Mettre les instructions qui changent peu en premier :**

```dockerfile
# D'abord les dépendances (changent peu)
COPY requirements.txt .
RUN pip install -r requirements.txt

# Ensuite le code (change souvent)
COPY . .
```

---

## Multi-stage builds

### Pourquoi multi-stage ?

- **Réduire la taille** : Image finale plus petite
- **Sécurité** : Exclure les outils de build
- **Performance** : Optimiser les builds

### Exemple

```dockerfile
# Stage 1 : Build
FROM python:3.11 as builder
WORKDIR /build
COPY requirements.txt .
RUN pip install --user -r requirements.txt

# Stage 2 : Runtime
FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY . .
ENV PATH=/root/.local/bin:$PATH
CMD ["python", "app.py"]
```

---

## 📊 Points clés à retenir

1. **Dockerfile** automatise la création d'images
2. **Instructions** : FROM, RUN, COPY, CMD
3. **Optimisation** : Réduire les couches
4. **Multi-stage** : Images plus légères
5. **.dockerignore** : Exclure des fichiers

## 🔗 Prochain module

Passer au module [5. Docker Compose](./05-docker-compose/README.md) pour orchestrer plusieurs conteneurs.

