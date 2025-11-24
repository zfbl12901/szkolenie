# 4. Index et Performance MongoDB

## 🎯 Objectifs

- Comprendre les index
- Créer différents types d'index
- Analyser les performances
- Optimiser les requêtes
- Utiliser explain()

## 📋 Table des matières

1. [Introduction aux index](#introduction-aux-index)
2. [Types d'index](#types-dindex)
3. [Créer des index](#créer-des-index)
4. [Analyser les performances](#analyser-les-performances)
5. [Optimisation](#optimisation)

---

## Introduction aux index

### Qu'est-ce qu'un index ?

**Index** = Structure de données pour accélérer les requêtes

- **Performance** : Recherche plus rapide
- **Coût** : Espace disque supplémentaire
- **Maintenance** : Mise à jour automatique
- **Similaire** : Index dans un livre

### Pourquoi des index ?

- **Recherche rapide** : Trouver rapidement
- **Tri rapide** : Trier efficacement
- **Unicité** : Garantir l'unicité
- **Performance** : Améliorer les requêtes

---

## Types d'index

### Index simple

```javascript
// Index sur un champ
db.users.createIndex({email: 1})

// 1 = croissant, -1 = décroissant
```

### Index composé

```javascript
// Index sur plusieurs champs
db.users.createIndex({name: 1, age: -1})
```

### Index unique

```javascript
// Garantir l'unicité
db.users.createIndex({email: 1}, {unique: true})
```

### Index texte

```javascript
// Pour recherche de texte
db.articles.createIndex({title: "text", content: "text"})
```

### Index géospatial

```javascript
// Pour données géographiques
db.places.createIndex({location: "2dsphere"})
```

---

## Créer des index

### Méthodes de création

```javascript
// Créer un index
db.collection.createIndex({field: 1})

// Créer avec options
db.collection.createIndex(
  {field: 1},
  {unique: true, sparse: true}
)

// Voir les index
db.collection.getIndexes()

// Supprimer un index
db.collection.dropIndex({field: 1})

// Supprimer tous les index (sauf _id)
db.collection.dropIndexes()
```

### Index par défaut

**Index _id :**
- Créé automatiquement
- Unique
- Ne peut pas être supprimé

---

## Analyser les performances

### explain()

**Voir le plan d'exécution :**

```javascript
// Plan d'exécution
db.users.find({email: "john@example.com"}).explain()

// Statistiques détaillées
db.users.find({email: "john@example.com"}).explain("executionStats")
```

### Métriques importantes

**executionStats :**
- **executionTimeMillis** : Temps d'exécution
- **totalDocsExamined** : Documents examinés
- **totalKeysExamined** : Clés examinées
- **nReturned** : Documents retournés

### Exemple

```javascript
// Sans index
db.users.find({email: "john@example.com"}).explain("executionStats")
// totalDocsExamined: 10000 (scan complet)

// Avec index
db.users.createIndex({email: 1})
db.users.find({email: "john@example.com"}).explain("executionStats")
// totalDocsExamined: 1 (utilisation de l'index)
```

---

## Optimisation

### Bonnes pratiques

**1. Indexer les champs fréquemment utilisés :**

```javascript
// Si souvent recherché par email
db.users.createIndex({email: 1})
```

**2. Index composé pour requêtes multiples :**

```javascript
// Si recherche par name ET age
db.users.createIndex({name: 1, age: 1})
```

**3. Éviter trop d'index :**

- Chaque index ralentit les écritures
- Utiliser seulement les index nécessaires

**4. Analyser les requêtes lentes :**

```javascript
// Activer le profiler
db.setProfilingLevel(1, {slowms: 100})

// Voir les requêtes lentes
db.system.profile.find().sort({ts: -1}).limit(10)
```

---

## Exemples pratiques

### Exemple 1 : Optimiser une requête

```javascript
// Requête lente
db.orders.find({customer: "John", status: "pending"})

// Créer un index composé
db.orders.createIndex({customer: 1, status: 1})

// Vérifier l'utilisation
db.orders.find({customer: "John", status: "pending"}).explain("executionStats")
```

### Exemple 2 : Index pour tri

```javascript
// Trier par date
db.sales.find().sort({date: -1})

// Créer un index pour le tri
db.sales.createIndex({date: -1})

// Vérifier
db.sales.find().sort({date: -1}).explain("executionStats")
```

---

## 📊 Points clés à retenir

1. **Index** : Accélèrent les recherches
2. **Types** : Simple, composé, unique, texte
3. **explain()** : Analyser les performances
4. **Optimisation** : Indexer les champs fréquents
5. **Équilibre** : Pas trop d'index

## 🔗 Prochain module

Passer au module [5. Modélisation des données](./05-data-modeling/README.md) pour apprendre à modéliser.

