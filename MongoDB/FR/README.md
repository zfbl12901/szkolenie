# Formation MongoDB pour Data Analyst

## 📚 Vue d'ensemble

Cette formation vous guide dans l'apprentissage de **MongoDB** en tant que Data Analyst. MongoDB est une base de données NoSQL orientée documents, idéale pour gérer des données non structurées et semi-structurées.

## 🎯 Objectifs pédagogiques

- Comprendre MongoDB et NoSQL
- Installer MongoDB
- Maîtriser les opérations CRUD
- Utiliser les requêtes et agrégations
- Optimiser avec les index
- Modéliser les données
- Intégrer MongoDB dans vos workflows
- Créer des projets pratiques pour votre portfolio

## 💰 Tout est gratuit !

Cette formation utilise uniquement :
- ✅ **MongoDB Community Server** : Gratuit et open-source
- ✅ **MongoDB Compass** : Interface graphique gratuite
- ✅ **MongoDB Atlas** : Cluster gratuit (512 MB)
- ✅ **Documentation officielle** : Guides complets gratuits
- ✅ **Tutoriels en ligne** : Ressources gratuites

**Budget total : 0€**

## 📖 Structure de la formation

### 1. [Prise en main MongoDB](./01-getting-started/README.md)
   - Installer MongoDB
   - Concepts de base
   - Premières opérations
   - Interface MongoDB Compass

### 2. [Opérations de base](./02-basic-operations/README.md)
   - CRUD (Create, Read, Update, Delete)
   - Collections et Documents
   - Types de données
   - Opérateurs de requête

### 3. [Requêtes et Agrégation](./03-queries-aggregation/README.md)
   - Requêtes avancées
   - Pipeline d'agrégation
   - Opérateurs d'agrégation
   - Groupement et calculs

### 4. [Index et Performance](./04-indexes-performance/README.md)
   - Créer des index
   - Types d'index
   - Analyse de performance
   - Optimisation des requêtes

### 5. [Modélisation des données](./05-data-modeling/README.md)
   - Modèles de données
   - Relations (Embedded vs References)
   - Schémas flexibles
   - Bonnes pratiques

### 6. [Fonctionnalités avancées](./06-advanced/README.md)
   - Transactions
   - Réplication
   - Sharding
   - Text Search

### 7. [Bonnes pratiques](./07-best-practices/README.md)
   - Sécurité
   - Performance
   - Maintenance
   - Backup et Restore

### 8. [Projets pratiques](./08-projets/README.md)
   - Application Python avec MongoDB
   - Pipeline de données
   - Analyse de données
   - Projets pour portfolio

## 🚀 Démarrage rapide

### Prérequis

- **Système d'exploitation** : Windows, Linux, ou macOS
- **4 Go RAM** : Minimum recommandé
- **Espace disque** : 5 Go libres

### Installation rapide

**Windows :**
1. Télécharger MongoDB : https://www.mongodb.com/try/download/community
2. Installer avec les options par défaut
3. Vérifier : `mongod --version`

**Linux :**
```bash
# Ubuntu/Debian
wget -qO - https://www.mongodb.org/static/pgp/server-7.0.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
sudo apt-get update
sudo apt-get install -y mongodb-org

# Démarrer MongoDB
sudo systemctl start mongod
sudo systemctl enable mongod
```

**macOS :**
```bash
# Avec Homebrew
brew tap mongodb/brew
brew install mongodb-community
brew services start mongodb-community
```

### Premier test

```bash
# Démarrer MongoDB
mongod

# Dans un autre terminal, se connecter
mongosh

# Tester
use test
db.collection.insertOne({name: "test"})
db.collection.find()
```

## 📊 Cas d'usage pour Data Analyst

- **Données non structurées** : JSON, logs, APIs
- **Flexibilité** : Schémas évolutifs
- **Agrégation** : Pipeline puissant pour l'analyse
- **Intégration** : Avec Python, R, PowerBI
- **Big Data** : Scalabilité horizontale

## 📚 Ressources gratuites

### Documentation officielle

- **MongoDB Documentation** : https://docs.mongodb.com/
- **MongoDB University** : https://university.mongodb.com/ (cours gratuits)
- **MongoDB Compass** : https://www.mongodb.com/products/compass

### Ressources externes

- **MongoDB Atlas** : Cluster gratuit 512 MB
- **YouTube** : Tutoriels MongoDB
- **GitHub** : Exemples MongoDB

## 🎓 Certifications (optionnel)

### MongoDB Certified Associate Developer

- **Coût** : ~$150
- **Préparation** : Documentation gratuite
- **Durée** : 1-2 mois
- **Niveau** : Intermédiaire

## 📝 Conventions

- Tous les exemples utilisent MongoDB 7.0+
- Les commandes fonctionnent avec `mongosh` (nouvelle CLI)
- Python avec `pymongo` pour les exemples
- Les données sont en format JSON

## 🤝 Contribution

Cette formation est conçue pour être évolutive. N'hésitez pas à proposer des améliorations.

## 📚 Ressources complémentaires

- [MongoDB Documentation](https://docs.mongodb.com/)
- [MongoDB University](https://university.mongodb.com/)
- [MongoDB Atlas](https://www.mongodb.com/cloud/atlas)
- [PyMongo Documentation](https://pymongo.readthedocs.io/)

