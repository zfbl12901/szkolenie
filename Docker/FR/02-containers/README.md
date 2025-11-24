# 2. Conteneurs Docker

## 🎯 Objectifs

- Créer et gérer des conteneurs
- Comprendre le cycle de vie
- Exécuter des commandes
- Gérer les logs
- Configurer les conteneurs

## 📋 Table des matières

1. [Cycle de vie d'un conteneur](#cycle-de-vie-dun-conteneur)
2. [Créer des conteneurs](#créer-des-conteneurs)
3. [Exécuter des commandes](#exécuter-des-commandes)
4. [Logs et débogage](#logs-et-débogage)
5. [Configuration](#configuration)

---

## Cycle de vie d'un conteneur

### États d'un conteneur

1. **Created** : Conteneur créé mais pas démarré
2. **Running** : Conteneur en cours d'exécution
3. **Paused** : Conteneur en pause
4. **Stopped** : Conteneur arrêté
5. **Removed** : Conteneur supprimé

### Commandes de cycle de vie

```bash
# Créer un conteneur
docker create --name my-container ubuntu

# Démarrer un conteneur
docker start my-container

# Arrêter un conteneur
docker stop my-container

# Redémarrer un conteneur
docker restart my-container

# Mettre en pause
docker pause my-container

# Reprendre
docker unpause my-container

# Supprimer un conteneur
docker rm my-container
```

---

## Créer des conteneurs

### Créer avec docker run

```bash
# Créer et démarrer un conteneur
docker run ubuntu echo "Hello"

# Créer sans démarrer
docker create --name my-container ubuntu

# Créer avec nom personnalisé
docker run --name my-app ubuntu
```

### Options importantes

```bash
# Mode interactif
docker run -it ubuntu bash

# Mode détaché (arrière-plan)
docker run -d nginx

# Exposer un port
docker run -p 8080:80 nginx

# Monter un volume
docker run -v /host/path:/container/path ubuntu

# Variables d'environnement
docker run -e MY_VAR=value ubuntu

# Nom du conteneur
docker run --name my-container ubuntu
```

---

## Exécuter des commandes

### Exécuter dans un conteneur en cours

```bash
# Exécuter une commande
docker exec my-container ls

# Mode interactif
docker exec -it my-container bash

# Exécuter Python
docker exec -it my-container python
```

### Exécuter au démarrage

```bash
# Commande par défaut
docker run ubuntu echo "Hello"

# Override la commande
docker run ubuntu ls -la

# Exécuter un script
docker run -v $(pwd):/app ubuntu bash /app/script.sh
```

---

## Logs et débogage

### Voir les logs

```bash
# Logs d'un conteneur
docker logs my-container

# Suivre les logs (tail -f)
docker logs -f my-container

# Dernières lignes
docker logs --tail 100 my-container

# Avec timestamp
docker logs -t my-container
```

### Inspecter un conteneur

```bash
# Informations complètes
docker inspect my-container

# Informations spécifiques
docker inspect --format='{{.State.Status}}' my-container

# Configuration réseau
docker inspect --format='{{.NetworkSettings.IPAddress}}' my-container
```

### Statistiques

```bash
# Statistiques en temps réel
docker stats

# Statistiques d'un conteneur
docker stats my-container

# Statistiques sans streaming
docker stats --no-stream
```

---

## Configuration

### Variables d'environnement

```bash
# Une variable
docker run -e MY_VAR=value ubuntu

# Plusieurs variables
docker run -e VAR1=value1 -e VAR2=value2 ubuntu

# Fichier .env
docker run --env-file .env ubuntu
```

### Ports

```bash
# Exposer un port
docker run -p 8080:80 nginx

# Exposer plusieurs ports
docker run -p 8080:80 -p 3306:3306 my-app

# Exposer tous les ports
docker run -P nginx
```

### Volumes

```bash
# Volume nommé
docker run -v my-volume:/data ubuntu

# Bind mount
docker run -v /host/path:/container/path ubuntu

# Volume anonyme
docker run -v /data ubuntu
```

---

## 📊 Points clés à retenir

1. **Cycle de vie** : Created → Running → Stopped → Removed
2. **docker run** : Crée et démarre
3. **docker exec** : Exécute dans un conteneur en cours
4. **docker logs** : Voir les logs
5. **Configuration** : Variables, ports, volumes

## 🔗 Prochain module

Passer au module [3. Images Docker](./03-images/README.md) pour apprendre à gérer les images.

