# 6. Fonctionnalités avancées MongoDB

## 🎯 Objectifs

- Utiliser les transactions
- Comprendre la réplication
- Maîtriser le sharding
- Utiliser la recherche de texte
- Fonctionnalités avancées

## 📋 Table des matières

1. [Transactions](#transactions)
2. [Réplication](#réplication)
3. [Sharding](#sharding)
4. [Text Search](#text-search)
5. [Autres fonctionnalités](#autres-fonctionnalités)

---

## Transactions

### Qu'est-ce qu'une transaction ?

**Transaction** = Groupe d'opérations atomiques

- **Atomique** : Tout ou rien
- **Cohérence** : Données cohérentes
- **Isolation** : Opérations isolées
- **Durabilité** : Changements persistants

### Utiliser les transactions

```javascript
// Démarrer une session
const session = db.getMongo().startSession()

// Démarrer une transaction
session.startTransaction()

try {
  // Opérations
  db.users.insertOne({name: "John"}, {session})
  db.orders.insertOne({user_id: "...", items: [...]}, {session})
  
  // Valider
  session.commitTransaction()
} catch (error) {
  // Annuler
  session.abortTransaction()
} finally {
  session.endSession()
}
```

---

## Réplication

### Qu'est-ce que la réplication ?

**Réplication** = Copies multiples des données

- **Haute disponibilité** : Pas de point de défaillance unique
- **Redondance** : Sauvegarde automatique
- **Performance** : Lecture depuis plusieurs serveurs

### Replica Set

**Configuration de base :**

```javascript
// 3 serveurs : 1 Primary + 2 Secondaries
// Primary : Écritures
// Secondaries : Lectures et backup
```

---

## Sharding

### Qu'est-ce que le sharding ?

**Sharding** = Partitionnement horizontal

- **Scalabilité** : Distribuer les données
- **Performance** : Traiter en parallèle
- **Stockage** : Plus de capacité

### Configuration

```javascript
// Shard key : Clé de partitionnement
db.collection.createIndex({shard_key: 1})

// Shard la collection
sh.shardCollection("mydb.mycollection", {shard_key: 1})
```

---

## Text Search

### Index de texte

```javascript
// Créer un index de texte
db.articles.createIndex({
  title: "text",
  content: "text"
})

// Rechercher
db.articles.find({
  $text: {$search: "mongodb tutorial"}
})

// Score de pertinence
db.articles.find(
  {$text: {$search: "mongodb"}},
  {score: {$meta: "textScore"}}
).sort({score: {$meta: "textScore"}})
```

---

## Autres fonctionnalités

### Validation de schéma

```javascript
// Définir un schéma de validation
db.createCollection("users", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["name", "email"],
      properties: {
        name: {
          bsonType: "string",
          description: "must be a string"
        },
        email: {
          bsonType: "string",
          pattern: "^.+@.+$"
        }
      }
    }
  }
})
```

### TTL Index

```javascript
// Index avec expiration automatique
db.sessions.createIndex(
  {created_at: 1},
  {expireAfterSeconds: 3600}  // Expire après 1 heure
)
```

---

## 📊 Points clés à retenir

1. **Transactions** : Opérations atomiques
2. **Réplication** : Haute disponibilité
3. **Sharding** : Scalabilité horizontale
4. **Text Search** : Recherche de texte
5. **Validation** : Schémas optionnels

## 🔗 Prochain module

Passer au module [7. Bonnes pratiques](./07-best-practices/README.md) pour les meilleures pratiques.

