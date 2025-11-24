# 3. Requêtes et Agrégation MongoDB

## 🎯 Objectifs

- Maîtriser les requêtes avancées
- Utiliser le pipeline d'agrégation
- Appliquer les opérateurs d'agrégation
- Effectuer des groupements et calculs
- Analyser des données complexes

## 📋 Table des matières

1. [Requêtes avancées](#requêtes-avancées)
2. [Pipeline d'agrégation](#pipeline-dagrégation)
3. [Opérateurs d'agrégation](#opérateurs-dagrégation)
4. [Groupement et calculs](#groupement-et-calculs)
5. [Exemples pratiques](#exemples-pratiques)

---

## Requêtes avancées

### Projection

```javascript
// Sélectionner des champs spécifiques
db.users.find({}, {name: 1, email: 1, _id: 0})

// Exclure des champs
db.users.find({}, {password: 0, secret: 0})
```

### Tri et limite

```javascript
// Trier par âge (croissant)
db.users.find().sort({age: 1})

// Trier par âge (décroissant)
db.users.find().sort({age: -1})

// Trier par plusieurs champs
db.users.find().sort({age: 1, name: 1})

// Limiter les résultats
db.users.find().limit(10)

// Sauter des résultats
db.users.find().skip(10).limit(10)
```

### Requêtes sur tableaux

```javascript
// Élément dans un tableau
db.users.find({hobbies: "reading"})

// Tous les éléments
db.users.find({hobbies: {$all: ["reading", "coding"]}})

// Taille du tableau
db.users.find({hobbies: {$size: 3}})

// Élément à une position
db.users.find({"hobbies.0": "reading"})
```

### Requêtes sur objets imbriqués

```javascript
// Accéder à un champ imbriqué
db.users.find({"address.city": "Paris"})

// Requête sur objet complet
db.users.find({address: {street: "123 Main St", city: "Paris"}})
```

---

## Pipeline d'agrégation

### Qu'est-ce qu'un pipeline ?

**Pipeline** = Série d'étapes de transformation

- **Étapes** : Chaque étape transforme les données
- **Séquentiel** : Résultat d'une étape = entrée de la suivante
- **Puissant** : Pour analyses complexes

### Structure de base

```javascript
db.collection.aggregate([
  { $match: { ... } },      // Filtrer
  { $group: { ... } },       // Grouper
  { $sort: { ... } },        // Trier
  { $project: { ... } }      // Sélectionner
])
```

---

## Opérateurs d'agrégation

### $match

**Filtrer les documents :**

```javascript
db.sales.aggregate([
  {$match: {amount: {$gt: 500}}}
])
```

### $group

**Grouper et calculer :**

```javascript
// Grouper par catégorie et calculer la somme
db.products.aggregate([
  {
    $group: {
      _id: "$category",
      total: {$sum: "$price"},
      count: {$sum: 1},
      average: {$avg: "$price"}
    }
  }
])
```

### $project

**Sélectionner et transformer :**

```javascript
db.users.aggregate([
  {
    $project: {
      name: 1,
      age: 1,
      isAdult: {$gte: ["$age", 18]}
    }
  }
])
```

### $sort

**Trier :**

```javascript
db.sales.aggregate([
  {$sort: {amount: -1}}
])
```

### $limit

**Limiter :**

```javascript
db.sales.aggregate([
  {$sort: {amount: -1}},
  {$limit: 10}
])
```

### $lookup

**Jointure (comme SQL JOIN) :**

```javascript
db.orders.aggregate([
  {
    $lookup: {
      from: "products",
      localField: "product_id",
      foreignField: "_id",
      as: "product_details"
    }
  }
])
```

---

## Groupement et calculs

### Opérateurs d'accumulation

```javascript
// Somme
{$sum: "$amount"}

// Moyenne
{$avg: "$price"}

// Minimum
{$min: "$price"}

// Maximum
{$max: "$price"}

// Premier
{$first: "$name"}

// Dernier
{$last: "$name"}

// Compter
{$sum: 1}
```

### Exemple : Analyse de ventes

```javascript
db.sales.aggregate([
  // Filtrer par date
  {
    $match: {
      date: {
        $gte: new Date("2024-01-01"),
        $lt: new Date("2024-02-01")
      }
    }
  },
  // Grouper par produit
  {
    $group: {
      _id: "$product",
      total_sales: {$sum: "$amount"},
      count: {$sum: 1},
      average: {$avg: "$amount"}
    }
  },
  // Trier par total
  {
    $sort: {total_sales: -1}
  },
  // Limiter aux 10 premiers
  {
    $limit: 10
  }
])
```

---

## Exemples pratiques

### Exemple 1 : Analyse de données utilisateurs

```javascript
db.users.aggregate([
  // Filtrer les utilisateurs actifs
  {
    $match: {active: true}
  },
  // Grouper par ville
  {
    $group: {
      _id: "$address.city",
      users: {$sum: 1},
      avgAge: {$avg: "$age"}
    }
  },
  // Trier par nombre d'utilisateurs
  {
    $sort: {users: -1}
  }
])
```

### Exemple 2 : Analyse de logs

```javascript
db.logs.aggregate([
  // Filtrer par type
  {
    $match: {type: "error"}
  },
  // Grouper par heure
  {
    $group: {
      _id: {
        $dateToString: {
          format: "%Y-%m-%d %H:00:00",
          date: "$timestamp"
        }
      },
      count: {$sum: 1}
    }
  },
  // Trier par date
  {
    $sort: {_id: 1}
  }
])
```

---

## 📊 Points clés à retenir

1. **Pipeline** : Série d'étapes de transformation
2. **$match** : Filtrer les documents
3. **$group** : Grouper et calculer
4. **$project** : Sélectionner et transformer
5. **Agrégation** : Puissant pour l'analyse

## 🔗 Prochain module

Passer au module [4. Index et Performance](./04-indexes-performance/README.md) pour optimiser les performances.

