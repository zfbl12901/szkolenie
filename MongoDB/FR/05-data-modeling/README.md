# 5. Modélisation des données MongoDB

## 🎯 Objectifs

- Comprendre les modèles de données
- Choisir entre Embedded et References
- Concevoir des schémas flexibles
- Appliquer les bonnes pratiques
- Optimiser la structure

## 📋 Table des matières

1. [Modèles de données](#modèles-de-données)
2. [Embedded vs References](#embedded-vs-references)
3. [Schémas flexibles](#schémas-flexibles)
4. [Bonnes pratiques](#bonnes-pratiques)
5. [Exemples pratiques](#exemples-pratiques)

---

## Modèles de données

### Modèle Embedded (Imbriqué)

**Tout dans un document :**

```javascript
// Utilisateur avec adresse imbriquée
{
  _id: ObjectId("..."),
  name: "John",
  email: "john@example.com",
  address: {
    street: "123 Main St",
    city: "Paris",
    zip: "75001"
  }
}
```

**Avantages :**
- Accès rapide (un seul document)
- Pas de jointure
- Données cohérentes

**Inconvénients :**
- Taille limitée (16 MB par document)
- Duplication possible

### Modèle References (Références)

**Documents séparés avec références :**

```javascript
// Collection users
{
  _id: ObjectId("..."),
  name: "John",
  email: "john@example.com"
}

// Collection addresses
{
  _id: ObjectId("..."),
  user_id: ObjectId("..."),
  street: "123 Main St",
  city: "Paris"
}
```

**Avantages :**
- Pas de limite de taille
- Pas de duplication
- Flexibilité

**Inconvénients :**
- Requiert des jointures ($lookup)
- Plus de requêtes

---

## Embedded vs References

### Quand utiliser Embedded ?

**Cas d'usage :**
- Données souvent accédées ensemble
- Petites quantités de données
- Relation 1:1 ou 1:peu
- Données qui changent rarement

**Exemple :**

```javascript
// Adresse d'un utilisateur (1:1)
{
  name: "John",
  address: {
    street: "123 Main St",
    city: "Paris"
  }
}
```

### Quand utiliser References ?

**Cas d'usage :**
- Grandes quantités de données
- Relation 1:beaucoup ou beaucoup:beaucoup
- Données partagées
- Données qui changent souvent

**Exemple :**

```javascript
// Articles d'un blog (1:beaucoup)
// Collection authors
{_id: ObjectId("..."), name: "John"}

// Collection articles
{
  _id: ObjectId("..."),
  title: "Article",
  author_id: ObjectId("...")
}
```

---

## Schémas flexibles

### Avantages de la flexibilité

**Évolution du schéma :**

```javascript
// Document initial
{
  name: "John",
  age: 30
}

// Ajouter un champ plus tard
{
  name: "John",
  age: 30,
  email: "john@example.com"  // Nouveau champ
}
```

### Gérer les variations

```javascript
// Documents avec structures différentes
db.products.insertMany([
  {name: "Laptop", price: 999, specs: {...}},
  {name: "Book", author: "Author", pages: 300},
  {name: "Service", duration: "1 hour", price: 50}
])
```

---

## Bonnes pratiques

### 1. Normalisation vs Dénormalisation

**Normalisation (SQL style) :**
- Données séparées
- Références
- Cohérence

**Dénormalisation (NoSQL style) :**
- Données dupliquées
- Accès rapide
- Performance

### 2. Patterns de modélisation

**One-to-Few :**
```javascript
// Embedded
{
  name: "John",
  addresses: [
    {street: "123 Main St"},
    {street: "456 Oak Ave"}
  ]
}
```

**One-to-Many :**
```javascript
// References
// Collection users
{_id: ObjectId("..."), name: "John"}

// Collection orders
{user_id: ObjectId("..."), items: [...]}
```

**Many-to-Many :**
```javascript
// References avec tableau
// Collection students
{_id: ObjectId("..."), courses: [ObjectId("..."), ObjectId("...")]}

// Collection courses
{_id: ObjectId("..."), students: [ObjectId("..."), ObjectId("...")]}
```

---

## Exemples pratiques

### Exemple 1 : E-commerce

```javascript
// Produit avec variantes (Embedded)
{
  _id: ObjectId("..."),
  name: "T-Shirt",
  price: 29.99,
  variants: [
    {size: "S", color: "Red", stock: 10},
    {size: "M", color: "Blue", stock: 15}
  ]
}

// Commandes (References)
// Collection orders
{
  _id: ObjectId("..."),
  user_id: ObjectId("..."),
  items: [
    {product_id: ObjectId("..."), quantity: 2}
  ]
}
```

### Exemple 2 : Blog

```javascript
// Article avec commentaires (Embedded pour récents)
{
  _id: ObjectId("..."),
  title: "Article",
  content: "...",
  comments: [
    {author: "User1", text: "Great!", date: new Date()}
  ]
}

// Auteurs (References)
// Collection authors
{_id: ObjectId("..."), name: "John"}

// Collection articles
{
  _id: ObjectId("..."),
  title: "Article",
  author_id: ObjectId("...")
}
```

---

## 📊 Points clés à retenir

1. **Embedded** : Pour données souvent accédées ensemble
2. **References** : Pour grandes quantités ou relations complexes
3. **Flexibilité** : Schémas évolutifs
4. **Patterns** : One-to-Few, One-to-Many, Many-to-Many
5. **Performance** : Équilibrer accès et cohérence

## 🔗 Prochain module

Passer au module [6. Fonctionnalités avancées](./06-advanced/README.md) pour approfondir.

