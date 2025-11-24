# 2. Opérations de base MongoDB

## 🎯 Objectifs

- Maîtriser les opérations CRUD
- Comprendre les collections et documents
- Utiliser les types de données
- Appliquer les opérateurs de requête
- Gérer les mises à jour

## 📋 Table des matières

1. [CRUD Operations](#crud-operations)
2. [Collections et Documents](#collections-et-documents)
3. [Types de données](#types-de-données)
4. [Opérateurs de requête](#opérateurs-de-requête)
5. [Mises à jour](#mises-à-jour)

---

## CRUD Operations

### Create (Créer)

**insertOne :**

```javascript
// Insérer un document
db.users.insertOne({
  name: "John",
  age: 30,
  email: "john@example.com"
})
```

**insertMany :**

```javascript
// Insérer plusieurs documents
db.users.insertMany([
  {name: "Alice", age: 28, email: "alice@example.com"},
  {name: "Bob", age: 32, email: "bob@example.com"},
  {name: "Charlie", age: 25, email: "charlie@example.com"}
])
```

### Read (Lire)

**find :**

```javascript
// Trouver tous les documents
db.users.find()

// Trouver avec filtre
db.users.find({age: 30})

// Trouver un seul document
db.users.findOne({name: "John"})

// Limiter les résultats
db.users.find().limit(5)

// Trier
db.users.find().sort({age: 1})  // 1 = ascendant, -1 = descendant
```

### Update (Mettre à jour)

**updateOne :**

```javascript
// Mettre à jour un document
db.users.updateOne(
  {name: "John"},
  {$set: {age: 31}}
)
```

**updateMany :**

```javascript
// Mettre à jour plusieurs documents
db.users.updateMany(
  {age: {$lt: 30}},
  {$set: {status: "young"}}
)
```

### Delete (Supprimer)

**deleteOne :**

```javascript
// Supprimer un document
db.users.deleteOne({name: "John"})
```

**deleteMany :**

```javascript
// Supprimer plusieurs documents
db.users.deleteMany({age: {$lt: 18}})
```

---

## Collections et Documents

### Créer une collection

```javascript
// Création automatique à la première insertion
db.mycollection.insertOne({test: "data"})

// Création explicite
db.createCollection("mycollection")

// Avec options
db.createCollection("mycollection", {
  capped: true,
  size: 100000,
  max: 5000
})
```

### Structure d'un document

```javascript
{
  _id: ObjectId("..."),  // Identifiant unique (auto-généré)
  name: "John",
  age: 30,
  address: {
    street: "123 Main St",
    city: "Paris",
    zip: "75001"
  },
  hobbies: ["reading", "coding", "traveling"],
  created_at: new Date()
}
```

---

## Types de données

### Types de base

```javascript
// String
{name: "John"}

// Number
{age: 30}
{price: 99.99}

// Boolean
{active: true}

// Date
{created_at: new Date()}
{birthday: new Date("1990-01-15")}

// Array
{hobbies: ["reading", "coding"]}

// Object (Embedded)
{address: {street: "123 Main St", city: "Paris"}}

// Null
{description: null}

// ObjectId
{_id: ObjectId("507f1f77bcf86cd799439011")}
```

### Exemple complet

```javascript
db.products.insertOne({
  name: "Laptop",
  price: 999.99,
  inStock: true,
  tags: ["electronics", "computers"],
  specifications: {
    cpu: "Intel i7",
    ram: "16GB",
    storage: "512GB SSD"
  },
  created_at: new Date()
})
```

---

## Opérateurs de requête

### Opérateurs de comparaison

```javascript
// Égal
db.users.find({age: 30})

// Plus grand que
db.users.find({age: {$gt: 30}})

// Plus grand ou égal
db.users.find({age: {$gte: 30}})

// Plus petit que
db.users.find({age: {$lt: 30}})

// Plus petit ou égal
db.users.find({age: {$lte: 30}})

// Différent
db.users.find({age: {$ne: 30}})

// Dans une liste
db.users.find({age: {$in: [25, 30, 35]}})

// Pas dans une liste
db.users.find({age: {$nin: [25, 30, 35]}})
```

### Opérateurs logiques

```javascript
// ET (AND)
db.users.find({
  $and: [
    {age: {$gt: 25}},
    {age: {$lt: 35}}
  ]
})

// OU (OR)
db.users.find({
  $or: [
    {age: {$lt: 25}},
    {age: {$gt: 35}}
  ]
})

// NON (NOT)
db.users.find({
  age: {$not: {$gt: 30}}
})
```

### Opérateurs de tableau

```javascript
// Contient un élément
db.users.find({hobbies: "reading"})

// Contient tous les éléments
db.users.find({hobbies: {$all: ["reading", "coding"]}})

// Taille du tableau
db.users.find({hobbies: {$size: 3}})
```

---

## Mises à jour

### Opérateurs de mise à jour

**$set :**

```javascript
// Définir un champ
db.users.updateOne(
  {name: "John"},
  {$set: {age: 31, city: "Lyon"}}
)
```

**$unset :**

```javascript
// Supprimer un champ
db.users.updateOne(
  {name: "John"},
  {$unset: {city: ""}}
)
```

**$inc :**

```javascript
// Incrémenter
db.users.updateOne(
  {name: "John"},
  {$inc: {age: 1}}
)
```

**$push :**

```javascript
// Ajouter à un tableau
db.users.updateOne(
  {name: "John"},
  {$push: {hobbies: "swimming"}}
)
```

**$pull :**

```javascript
// Retirer d'un tableau
db.users.updateOne(
  {name: "John"},
  {$pull: {hobbies: "swimming"}}
)
```

---

## Exemples pratiques

### Exemple 1 : Gestion de commandes

```javascript
use shopdb

// Créer une commande
db.orders.insertOne({
  order_id: "ORD001",
  customer: "John Doe",
  items: [
    {product: "Laptop", quantity: 1, price: 999},
    {product: "Mouse", quantity: 2, price: 25}
  ],
  total: 1049,
  status: "pending",
  created_at: new Date()
})

// Trouver les commandes en attente
db.orders.find({status: "pending"})

// Mettre à jour le statut
db.orders.updateOne(
  {order_id: "ORD001"},
  {$set: {status: "completed"}}
)
```

### Exemple 2 : Analyse de données

```javascript
use analyticsdb

// Insérer des événements
db.events.insertMany([
  {event: "page_view", user: "user1", timestamp: new Date()},
  {event: "click", user: "user1", timestamp: new Date()},
  {event: "page_view", user: "user2", timestamp: new Date()}
])

// Trouver les événements d'un utilisateur
db.events.find({user: "user1"})

// Compter les événements
db.events.countDocuments({event: "page_view"})
```

---

## 📊 Points clés à retenir

1. **CRUD** : Create, Read, Update, Delete
2. **Documents** : Format JSON flexible
3. **Collections** : Groupes de documents
4. **Opérateurs** : Pour filtrer et mettre à jour
5. **Types** : Données variées supportées

## 🔗 Prochain module

Passer au module [3. Requêtes et Agrégation](./03-queries-aggregation/README.md) pour les requêtes avancées.

