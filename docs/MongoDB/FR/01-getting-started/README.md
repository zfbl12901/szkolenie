# 1. Prise en main MongoDB

## 🎯 Objectifs

- Comprendre MongoDB et NoSQL
- Installer MongoDB
- Comprendre les concepts de base
- Utiliser MongoDB Compass
- Premières opérations

## 📋 Table des matières

1. [Introduction à MongoDB](#introduction-à-mongodb)
2. [Installation](#installation)
3. [Concepts de base](#concepts-de-base)
4. [MongoDB Compass](#mongodb-compass)
5. [Premières opérations](#premières-opérations)

---

## Introduction à MongoDB

### Qu'est-ce que MongoDB ?

**MongoDB** = Base de données NoSQL orientée documents

- **NoSQL** : Non relationnel
- **Documents** : Stockage en format JSON (BSON)
- **Flexible** : Schémas évolutifs
- **Scalable** : Scalabilité horizontale
- **Open-source** : Gratuit et open-source

### Pourquoi MongoDB pour Data Analyst ?

- **Données non structurées** : JSON, logs, APIs
- **Flexibilité** : Schémas qui évoluent
- **Agrégation** : Pipeline puissant pour l'analyse
- **Intégration** : Avec Python, R, PowerBI
- **Performance** : Rapide pour les requêtes complexes

### MongoDB vs SQL

**MongoDB (NoSQL) :**
- Documents JSON
- Schéma flexible
- Scalabilité horizontale
- Idéal pour données non structurées

**SQL (Relationnel) :**
- Tables structurées
- Schéma fixe
- Relations complexes
- Idéal pour données structurées

---

## Installation

### Windows

**Étape 1 : Télécharger MongoDB**

1. Aller sur : https://www.mongodb.com/try/download/community
2. Sélectionner Windows
3. Télécharger l'installateur MSI
4. Exécuter l'installateur
5. Choisir "Complete" installation

**Étape 2 : Vérifier l'installation**

```bash
# Vérifier la version
mongod --version

# Démarrer MongoDB
mongod
```

**Étape 3 : Installer MongoDB Compass (optionnel)**

1. Télécharger : https://www.mongodb.com/products/compass
2. Installer avec l'installateur

### Linux

**Ubuntu/Debian :**

```bash
# Importer la clé GPG
wget -qO - https://www.mongodb.org/static/pgp/server-7.0.asc | sudo apt-key add -

# Ajouter le repository
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

# Installer
sudo apt-get update
sudo apt-get install -y mongodb-org

# Démarrer MongoDB
sudo systemctl start mongod
sudo systemctl enable mongod

# Vérifier
sudo systemctl status mongod
```

### macOS

**Avec Homebrew :**

```bash
# Ajouter le tap
brew tap mongodb/brew

# Installer MongoDB
brew install mongodb-community

# Démarrer MongoDB
brew services start mongodb-community

# Vérifier
brew services list
```

---

## Concepts de base

### Base de données (Database)

**Base de données** = Conteneur de collections

- **Création automatique** : Créée à la première utilisation
- **Nom** : Identifiant unique
- **Collections** : Contient des collections

### Collection

**Collection** = Groupe de documents

- **Équivalent** : Table en SQL
- **Flexible** : Pas de schéma imposé
- **Documents** : Contient des documents

### Document

**Document** = Enregistrement en format JSON

- **Format** : BSON (Binary JSON)
- **Flexible** : Structure variable
- **Champs** : Paires clé-valeur

### Exemple de structure

```
Database: mydb
  └── Collection: users
        ├── Document 1: {_id: 1, name: "John", age: 30}
        ├── Document 2: {_id: 2, name: "Jane", age: 25}
        └── Document 3: {_id: 3, name: "Bob", age: 35}
```

---

## MongoDB Compass

### Qu'est-ce que Compass ?

**MongoDB Compass** = Interface graphique

- **Visualisation** : Voir les données
- **Requêtes** : Exécuter des requêtes
- **Analyse** : Analyser les performances
- **Gestion** : Gérer les index

### Installation

1. Télécharger : https://www.mongodb.com/products/compass
2. Installer
3. Lancer Compass
4. Se connecter à `mongodb://localhost:27017`

### Utilisation de base

**Se connecter :**
- Host : `localhost`
- Port : `27017`
- Pas d'authentification (par défaut)

**Naviguer :**
- Voir les bases de données
- Voir les collections
- Voir les documents

---

## Premières opérations

### Se connecter avec mongosh

```bash
# Lancer mongosh
mongosh

# Voir les bases de données
show dbs

# Utiliser une base de données
use mydb

# Voir les collections
show collections

# Insérer un document
db.users.insertOne({name: "John", age: 30, city: "Paris"})

# Trouver des documents
db.users.find()

# Trouver un document spécifique
db.users.findOne({name: "John"})
```

### Exemple complet

```javascript
// Se connecter
mongosh

// Utiliser une base de données
use testdb

// Insérer plusieurs documents
db.products.insertMany([
  {name: "Laptop", price: 999, category: "Electronics"},
  {name: "Book", price: 19, category: "Education"},
  {name: "Phone", price: 699, category: "Electronics"}
])

// Trouver tous les produits
db.products.find()

// Trouver par catégorie
db.products.find({category: "Electronics"})

// Compter les documents
db.products.countDocuments()
```

---

## Commandes essentielles

### Gestion des bases de données

```javascript
// Voir les bases de données
show dbs

// Utiliser une base de données
use mydb

// Voir la base de données actuelle
db

// Supprimer une base de données
db.dropDatabase()
```

### Gestion des collections

```javascript
// Voir les collections
show collections

// Créer une collection (automatique à l'insertion)
db.mycollection.insertOne({test: "data"})

// Supprimer une collection
db.mycollection.drop()

// Renommer une collection
db.mycollection.renameCollection("newcollection")
```

---

## Exemples pratiques

### Exemple 1 : Gestion d'utilisateurs

```javascript
use userdb

// Insérer des utilisateurs
db.users.insertMany([
  {name: "Alice", email: "alice@example.com", age: 28},
  {name: "Bob", email: "bob@example.com", age: 32},
  {name: "Charlie", email: "charlie@example.com", age: 25}
])

// Trouver tous les utilisateurs
db.users.find()

// Trouver les utilisateurs de plus de 30 ans
db.users.find({age: {$gt: 30}})
```

### Exemple 2 : Données de ventes

```javascript
use salesdb

// Insérer des ventes
db.sales.insertMany([
  {product: "Laptop", amount: 999, date: new Date("2024-01-15")},
  {product: "Phone", amount: 699, date: new Date("2024-01-16")},
  {product: "Tablet", amount: 399, date: new Date("2024-01-17")}
])

// Trouver toutes les ventes
db.sales.find()

// Trouver les ventes supérieures à 500
db.sales.find({amount: {$gt: 500}})
```

---

## Dépannage

### Problème : MongoDB ne démarre pas

**Solutions :**
1. Vérifier les logs : `/var/log/mongodb/mongod.log` (Linux)
2. Vérifier les permissions
3. Vérifier que le port 27017 est libre
4. Redémarrer le service : `sudo systemctl restart mongod`

### Problème : Connexion refusée

**Solutions :**
1. Vérifier que MongoDB est démarré
2. Vérifier le port : `netstat -an | grep 27017`
3. Vérifier le firewall

---

## 📊 Points clés à retenir

1. **MongoDB** = Base de données NoSQL orientée documents
2. **Documents** = Format JSON (BSON)
3. **Collections** = Groupes de documents
4. **Bases de données** = Conteneurs de collections
5. **Compass** = Interface graphique

## 🔗 Prochain module

Passer au module [2. Opérations de base](./02-basic-operations/README.md) pour maîtriser le CRUD.

