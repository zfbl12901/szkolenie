# 7. Bonnes pratiques MongoDB

## 🎯 Objectifs

- Sécurité
- Performance
- Maintenance
- Backup et Restore
- Monitoring

## 📋 Table des matières

1. [Sécurité](#sécurité)
2. [Performance](#performance)
3. [Maintenance](#maintenance)
4. [Backup et Restore](#backup-et-restore)
5. [Monitoring](#monitoring)

---

## Sécurité

### Authentification

```javascript
// Créer un utilisateur admin
use admin
db.createUser({
  user: "admin",
  pwd: "secure_password",
  roles: ["root"]
})

// Créer un utilisateur pour une base
use mydb
db.createUser({
  user: "app_user",
  pwd: "app_password",
  roles: ["readWrite"]
})
```

### Connexion sécurisée

```bash
# Se connecter avec authentification
mongosh -u admin -p secure_password --authenticationDatabase admin
```

### Bonnes pratiques sécurité

- **Authentification** : Toujours activer
- **Autorisation** : Principe du moindre privilège
- **Chiffrement** : Pour données sensibles
- **Réseau** : Limiter l'accès réseau

---

## Performance

### Index

```javascript
// Indexer les champs de recherche fréquents
db.users.createIndex({email: 1})

// Index composé pour requêtes multiples
db.orders.createIndex({customer: 1, date: -1})
```

### Requêtes

```javascript
// Utiliser projection pour limiter les données
db.users.find({}, {name: 1, email: 1})

// Limiter les résultats
db.users.find().limit(100)

// Éviter les scans complets
// Toujours utiliser des index
```

### Write Concern

```javascript
// Contrôler la confirmation d'écriture
db.collection.insertOne(
  {data: "value"},
  {writeConcern: {w: 1, j: true}}
)
```

---

## Maintenance

### Nettoyage

```javascript
// Supprimer les documents obsolètes
db.logs.deleteMany({
  created_at: {$lt: new Date("2024-01-01")}
})

// Compacter la collection
db.runCommand({compact: "collection_name"})
```

### Statistiques

```javascript
// Statistiques d'une collection
db.collection.stats()

// Statistiques de la base
db.stats()
```

---

## Backup et Restore

### Backup (mongodump)

```bash
# Backup d'une base de données
mongodump --db mydb --out /backup/

# Backup d'une collection
mongodump --db mydb --collection users --out /backup/
```

### Restore (mongorestore)

```bash
# Restaurer une base de données
mongorestore --db mydb /backup/mydb/

# Restaurer une collection
mongorestore --db mydb --collection users /backup/mydb/users.bson
```

---

## Monitoring

### Server Status

```javascript
// Statut du serveur
db.serverStatus()

// Informations sur les opérations
db.currentOp()

// Statistiques de réplication
rs.status()
```

---

## 📊 Points clés à retenir

1. **Sécurité** : Authentification et autorisation
2. **Performance** : Index et requêtes optimisées
3. **Maintenance** : Nettoyage régulier
4. **Backup** : Sauvegardes régulières
5. **Monitoring** : Surveiller les performances

## 🔗 Prochain module

Passer au module [8. Projets pratiques](./08-projets/README.md) pour créer des projets complets.

