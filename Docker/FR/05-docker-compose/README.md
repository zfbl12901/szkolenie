# 5. Docker Compose

## 🎯 Objectifs

- Comprendre Docker Compose
- Orchestrer plusieurs conteneurs
- Créer des fichiers docker-compose.yml
- Gérer les services et réseaux
- Utiliser les variables d'environnement

## 📋 Table des matières

1. [Introduction à Docker Compose](#introduction-à-docker-compose)
2. [Fichier docker-compose.yml](#fichier-docker-composeyml)
3. [Services](#services)
4. [Réseaux et Volumes](#réseaux-et-volumes)
5. [Commandes](#commandes)

---

## Introduction à Docker Compose

### Qu'est-ce que Docker Compose ?

**Docker Compose** = Outil pour orchestrer plusieurs conteneurs

- **Multi-conteneurs** : Gère plusieurs conteneurs
- **Configuration** : Fichier YAML simple
- **Orchestration** : Démarre/arrête tous les services
- **Réseaux** : Crée automatiquement des réseaux

### Pourquoi Docker Compose ?

- **Simplicité** : Un fichier pour tout
- **Reproductibilité** : Même environnement partout
- **Développement** : Stack complète locale
- **Production** : Déploiement simplifié

---

## Fichier docker-compose.yml

### Structure de base

```yaml
version: '3.8'

services:
  web:
    image: nginx
    ports:
      - "8080:80"
  
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: password
```

### Exemple complet

```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8080:8000"
    environment:
      - DATABASE_URL=postgresql://db:5432/mydb
    depends_on:
      - db
  
  db:
    image: postgres:15
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

---

## Services

### Définir un service

```yaml
services:
  app:
    image: python:3.11
    command: python app.py
    working_dir: /app
    volumes:
      - .:/app
```

### Build vs Image

```yaml
# Utiliser une image existante
services:
  web:
    image: nginx:latest

# Construire depuis Dockerfile
services:
  app:
    build: .
    # ou
    build:
      context: .
      dockerfile: Dockerfile.prod
```

### Dépendances

```yaml
services:
  app:
    depends_on:
      - db
      - redis
  
  db:
    image: postgres
  
  redis:
    image: redis
```

---

## Réseaux et Volumes

### Réseaux

```yaml
services:
  app:
    networks:
      - frontend
      - backend
  
  db:
    networks:
      - backend

networks:
  frontend:
  backend:
```

### Volumes

```yaml
services:
  db:
    volumes:
      - db-data:/var/lib/postgresql/data
      - ./backup:/backup

volumes:
  db-data:
```

---

## Commandes

### Démarrer les services

```bash
# Démarrer tous les services
docker-compose up

# En arrière-plan
docker-compose up -d

# Reconstruire les images
docker-compose up --build
```

### Arrêter les services

```bash
# Arrêter les services
docker-compose stop

# Arrêter et supprimer
docker-compose down

# Supprimer avec volumes
docker-compose down -v
```

### Gestion des services

```bash
# Voir les services en cours
docker-compose ps

# Voir les logs
docker-compose logs

# Logs d'un service
docker-compose logs web

# Exécuter une commande
docker-compose exec web bash

# Redémarrer un service
docker-compose restart web
```

---

## 📊 Points clés à retenir

1. **Docker Compose** orchestre plusieurs conteneurs
2. **docker-compose.yml** définit la configuration
3. **Services** sont les conteneurs
4. **Réseaux et Volumes** pour la communication et données
5. **Commandes** : up, down, logs, exec

## 🔗 Prochain module

Passer au module [6. Volumes et Réseaux](./06-volumes-networks/README.md) pour approfondir.

