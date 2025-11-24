# Formation Docker pour Data Analyst

## 📚 Vue d'ensemble

Cette formation vous guide dans l'apprentissage de **Docker** en tant que Data Analyst. Docker est une plateforme de conteneurisation qui permet de créer, déployer et exécuter des applications dans des conteneurs isolés.

## 🎯 Objectifs pédagogiques

- Comprendre Docker et la conteneurisation
- Installer Docker
- Créer et gérer des conteneurs
- Construire des images Docker
- Utiliser Docker Compose
- Intégrer Docker dans vos workflows de données
- Créer des projets pratiques pour votre portfolio

## 💰 Tout est gratuit !

Cette formation utilise uniquement :
- ✅ **Docker Desktop** : Gratuit pour usage personnel/éducation
- ✅ **Docker Hub** : Registre public gratuit
- ✅ **Documentation officielle** : Guides complets gratuits
- ✅ **Tutoriels en ligne** : Ressources gratuites

**Budget total : 0€**

## 📖 Structure de la formation

### 1. [Prise en main Docker](./01-getting-started/README.md)
   - Installer Docker
   - Concepts de base
   - Premiers conteneurs
   - Commandes essentielles

### 2. [Conteneurs](./02-containers/README.md)
   - Créer des conteneurs
   - Gérer le cycle de vie
   - Exécuter des commandes
   - Logs et débogage

### 3. [Images Docker](./03-images/README.md)
   - Comprendre les images
   - Télécharger des images
   - Créer des images personnalisées
   - Gérer les images

### 4. [Dockerfile](./04-dockerfile/README.md)
   - Écrire un Dockerfile
   - Bonnes pratiques
   - Optimisation des images
   - Multi-stage builds

### 5. [Docker Compose](./05-docker-compose/README.md)
   - Orchestrer plusieurs conteneurs
   - Fichier docker-compose.yml
   - Services et réseaux
   - Variables d'environnement

### 6. [Volumes et Réseaux](./06-volumes-networks/README.md)
   - Gérer les volumes
   - Créer des réseaux
   - Partager des données
   - Persistance des données

### 7. [Bonnes pratiques](./07-best-practices/README.md)
   - Sécurité
   - Performance
   - Organisation
   - Maintenance

### 8. [Projets pratiques](./08-projets/README.md)
   - Conteneuriser une application Python
   - Pipeline de données avec Docker
   - Stack complète avec Docker Compose
   - Projets pour portfolio

## 🚀 Démarrage rapide

### Prérequis

- **Système d'exploitation** : Windows, Linux, ou macOS
- **4 Go RAM** : Minimum recommandé
- **Espace disque** : 20 Go libres

### Installation rapide

**Windows/Mac :**
1. Télécharger Docker Desktop : https://www.docker.com/products/docker-desktop
2. Installer et lancer Docker Desktop
3. Vérifier l'installation : `docker --version`

**Linux :**
```bash
# Installer Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Démarrer Docker
sudo systemctl start docker
sudo systemctl enable docker

# Vérifier
docker --version
```

### Premier conteneur

```bash
# Exécuter un conteneur Hello World
docker run hello-world

# Exécuter un conteneur interactif
docker run -it ubuntu bash
```

## 📊 Cas d'usage pour Data Analyst

- **Environnements reproductibles** : Même environnement partout
- **Isolation** : Séparer les dépendances
- **Déploiement** : Déployer facilement des applications
- **CI/CD** : Intégrer dans les pipelines
- **Data Science** : Environnements Python/R isolés

## 📚 Ressources gratuites

### Documentation officielle

- **Docker Documentation** : https://docs.docker.com/
  - Guides complets
- **Docker Hub** : https://hub.docker.com/
  - Images publiques
- **Docker Playground** : https://labs.play-with-docker.com/
  - Environnement de test en ligne

### Ressources externes

- **YouTube** : Tutoriels Docker
- **GitHub** : Exemples Docker
- **Stack Overflow** : Questions et réponses

## 🎓 Certifications (optionnel)

### Docker Certified Associate (DCA)

- **Coût** : ~$195
- **Préparation** : Documentation gratuite
- **Durée** : 2-4 semaines
- **Niveau** : Intermédiaire

## 📝 Conventions

- Tous les exemples sont testés sur Docker Desktop
- Les commandes fonctionnent sur Windows, Linux, et macOS
- Les chemins peuvent varier selon le système

## 🤝 Contribution

Cette formation est conçue pour être évolutive. N'hésitez pas à proposer des améliorations.

## 📚 Ressources complémentaires

- [Documentation Docker](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/)
- [Docker Playground](https://labs.play-with-docker.com/)
- [Docker GitHub](https://github.com/docker)

