# 7. Bonnes pratiques Docker

## 🎯 Objectifs

- Sécuriser les conteneurs
- Optimiser les performances
- Organiser les projets
- Maintenir les images
- Gérer les ressources

## 📋 Table des matières

1. [Sécurité](#sécurité)
2. [Performance](#performance)
3. [Organisation](#organisation)
4. [Maintenance](#maintenance)
5. [Ressources](#ressources)

---

## Sécurité

### Utiliser des images officielles

```dockerfile
# Bon
FROM python:3.11-slim

# Éviter
FROM random-user/python:latest
```

### Ne pas utiliser root

```dockerfile
# Créer un utilisateur non-root
RUN useradd -m appuser
USER appuser
```

### Limiter les privilèges

```bash
# Ne pas utiliser --privileged
docker run --privileged my-container  # Éviter

# Utiliser des capabilities spécifiques si nécessaire
docker run --cap-add NET_ADMIN my-container
```

### Secrets

```yaml
# docker-compose.yml
services:
  app:
    secrets:
      - db_password

secrets:
  db_password:
    file: ./secrets/password.txt
```

---

## Performance

### Utiliser .dockerignore

```
__pycache__
*.pyc
.git
.env
node_modules
*.log
```

### Optimiser les couches

```dockerfile
# Mauvais
RUN apt update
RUN apt install -y python3
RUN apt clean

# Bon
RUN apt update && \
    apt install -y python3 && \
    apt clean
```

### Cache des builds

**Ordre des instructions :**

```dockerfile
# D'abord les dépendances (changent peu)
COPY requirements.txt .
RUN pip install -r requirements.txt

# Ensuite le code (change souvent)
COPY . .
```

---

## Organisation

### Structure de projet

```
my-project/
├── Dockerfile
├── docker-compose.yml
├── .dockerignore
├── requirements.txt
├── src/
│   └── app.py
└── data/
    └── data.csv
```

### Tags d'images

```bash
# Tags sémantiques
docker build -t my-app:1.0.0 .
docker build -t my-app:latest .

# Tags pour environnement
docker build -t my-app:dev .
docker build -t my-app:prod .
```

---

## Maintenance

### Nettoyer les ressources

```bash
# Supprimer les conteneurs arrêtés
docker container prune

# Supprimer les images non utilisées
docker image prune

# Supprimer les volumes non utilisés
docker volume prune

# Nettoyer tout
docker system prune -a
```

### Mettre à jour les images

```bash
# Mettre à jour une image
docker pull python:3.11

# Reconstruire
docker-compose build --no-cache
docker-compose up
```

---

## Ressources

### Limiter les ressources

```yaml
# docker-compose.yml
services:
  app:
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M
```

---

## 📊 Points clés à retenir

1. **Sécurité** : Images officielles, non-root
2. **Performance** : Optimiser les couches, cache
3. **Organisation** : Structure claire, tags
4. **Maintenance** : Nettoyer régulièrement
5. **Ressources** : Limiter l'utilisation

## 🔗 Prochain module

Passer au module [8. Projets pratiques](./08-projets/README.md) pour créer des projets complets.

