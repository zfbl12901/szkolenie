# 6. Volumes et Réseaux Docker

## 🎯 Objectifs

- Comprendre les volumes Docker
- Gérer la persistance des données
- Créer et gérer des réseaux
- Partager des données entre conteneurs
- Configurer la communication réseau

## 📋 Table des matières

1. [Volumes](#volumes)
2. [Bind Mounts](#bind-mounts)
3. [Réseaux](#réseaux)
4. [Communication entre conteneurs](#communication-entre-conteneurs)
5. [Exemples pratiques](#exemples-pratiques)

---

## Volumes

### Qu'est-ce qu'un volume ?

**Volume** = Stockage persistant pour les données

- **Persistant** : Survit à la suppression du conteneur
- **Géré par Docker** : Stocké dans `/var/lib/docker/volumes`
- **Partageable** : Plusieurs conteneurs peuvent l'utiliser
- **Performant** : Plus rapide que bind mounts

### Créer un volume

```bash
# Créer un volume
docker volume create my-volume

# Lister les volumes
docker volume ls

# Inspecter un volume
docker volume inspect my-volume

# Supprimer un volume
docker volume rm my-volume
```

### Utiliser un volume

```bash
# Volume nommé
docker run -v my-volume:/data ubuntu

# Volume anonyme
docker run -v /data ubuntu

# Dans docker-compose.yml
volumes:
  - my-volume:/data
```

---

## Bind Mounts

### Qu'est-ce qu'un bind mount ?

**Bind Mount** = Lien direct vers un répertoire hôte

- **Direct** : Accès direct aux fichiers hôte
- **Développement** : Idéal pour le développement
- **Performance** : Dépend du système de fichiers hôte

### Utiliser un bind mount

```bash
# Bind mount
docker run -v /host/path:/container/path ubuntu

# Avec docker-compose.yml
volumes:
  - ./data:/app/data
  - /absolute/path:/container/path
```

### Différences : Volume vs Bind Mount

**Volume :**
- Géré par Docker
- Meilleure performance
- Portable
- Recommandé pour production

**Bind Mount :**
- Lien direct
- Accès direct
- Dépend du système hôte
- Recommandé pour développement

---

## Réseaux

### Types de réseaux

1. **Bridge** : Réseau par défaut (isolation)
2. **Host** : Utilise le réseau hôte
3. **None** : Pas de réseau
4. **Overlay** : Pour Docker Swarm

### Créer un réseau

```bash
# Créer un réseau
docker network create my-network

# Lister les réseaux
docker network ls

# Inspecter un réseau
docker network inspect my-network

# Supprimer un réseau
docker network rm my-network
```

### Connecter un conteneur

```bash
# Connecter au démarrage
docker run --network my-network ubuntu

# Connecter un conteneur existant
docker network connect my-network container-id

# Déconnecter
docker network disconnect my-network container-id
```

---

## Communication entre conteneurs

### Même réseau

```bash
# Créer un réseau
docker network create app-network

# Conteneur 1
docker run --name app --network app-network my-app

# Conteneur 2 (peut communiquer avec app)
docker run --name db --network app-network postgres
```

### Avec Docker Compose

```yaml
services:
  app:
    networks:
      - app-network
  
  db:
    networks:
      - app-network

networks:
  app-network:
```

### Résolution DNS

**Les conteneurs peuvent se trouver par nom :**

```python
# Dans le conteneur app
import psycopg2
conn = psycopg2.connect(
    host="db",  # Nom du service
    database="mydb"
)
```

---

## Exemples pratiques

### Exemple 1 : Base de données avec volume

```yaml
version: '3.8'

services:
  db:
    image: postgres:15
    volumes:
      - db-data:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: mydb

volumes:
  db-data:
```

### Exemple 2 : Application avec bind mount

```yaml
version: '3.8'

services:
  app:
    build: .
    volumes:
      - ./src:/app/src  # Développement
    networks:
      - app-network

networks:
  app-network:
```

---

## 📊 Points clés à retenir

1. **Volumes** pour persistance gérée par Docker
2. **Bind Mounts** pour accès direct
3. **Réseaux** pour communication
4. **DNS** : Résolution par nom de service
5. **Docker Compose** simplifie la gestion

## 🔗 Prochain module

Passer au module [7. Bonnes pratiques](./07-best-practices/README.md) pour les meilleures pratiques.

