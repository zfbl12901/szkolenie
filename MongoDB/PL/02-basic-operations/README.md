# 2. Operacje podstawowe MongoDB

## 🎯 Cele

- Opanować operacje CRUD
- Zrozumieć kolekcje i dokumenty
- Używać typów danych
- Stosować operatory zapytań
- Zarządzać aktualizacjami

## 📋 Spis treści

1. [Operacje CRUD](#operacje-crud)
2. [Kolekcje i Dokumenty](#kolekcje-i-dokumenty)
3. [Typy danych](#typy-danych)
4. [Operatory zapytań](#operatory-zapytań)
5. [Aktualizacje](#aktualizacje)

---

## Operacje CRUD

### Create (Tworzyć)

```javascript
// Wstawić jeden dokument
db.users.insertOne({
  name: "John",
  age: 30,
  email: "john@example.com"
})

// Wstawić wiele dokumentów
db.users.insertMany([
  {name: "Alice", age: 28},
  {name: "Bob", age: 32}
])
```

### Read (Czytać)

```javascript
// Znaleźć wszystkie dokumenty
db.users.find()

// Znaleźć z filtrem
db.users.find({age: 30})

// Znaleźć jeden dokument
db.users.findOne({name: "John"})

// Ograniczyć wyniki
db.users.find().limit(5)

// Sortować
db.users.find().sort({age: 1})
```

### Update (Aktualizować)

```javascript
// Aktualizować jeden dokument
db.users.updateOne(
  {name: "John"},
  {$set: {age: 31}}
)

// Aktualizować wiele dokumentów
db.users.updateMany(
  {age: {$lt: 30}},
  {$set: {status: "young"}}
)
```

### Delete (Usuwać)

```javascript
// Usunąć jeden dokument
db.users.deleteOne({name: "John"})

// Usunąć wiele dokumentów
db.users.deleteMany({age: {$lt: 18}})
```

---

## Typy danych

### Typy podstawowe

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

// Array
{hobbies: ["reading", "coding"]}

// Object (Zagnieżdżony)
{address: {street: "123 Main St", city: "Warsaw"}}
```

---

## Operatory zapytań

### Operatory porównania

```javascript
// Równy
db.users.find({age: 30})

// Większy niż
db.users.find({age: {$gt: 30}})

// Mniejszy niż
db.users.find({age: {$lt: 30}})

// W liście
db.users.find({age: {$in: [25, 30, 35]}})
```

### Operatory logiczne

```javascript
// I (AND)
db.users.find({
  $and: [
    {age: {$gt: 25}},
    {age: {$lt: 35}}
  ]
})

// LUB (OR)
db.users.find({
  $or: [
    {age: {$lt: 25}},
    {age: {$gt: 35}}
  ]
})
```

---

## Aktualizacje

### Operatory aktualizacji

```javascript
// $set: Ustawić pole
db.users.updateOne(
  {name: "John"},
  {$set: {age: 31}}
)

// $inc: Zwiększyć
db.users.updateOne(
  {name: "John"},
  {$inc: {age: 1}}
)

// $push: Dodać do tablicy
db.users.updateOne(
  {name: "John"},
  {$push: {hobbies: "swimming"}}
)
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **CRUD** : Create, Read, Update, Delete
2. **Dokumenty** : Elastyczny format JSON
3. **Kolekcje** : Grupy dokumentów
4. **Operatory** : Do filtrowania i aktualizacji
5. **Typy** : Różne typy danych wspierane

## 🔗 Następny moduł

Przejdź do modułu [3. Zapytania i Agregacja](./03-queries-aggregation/README.md), aby poznać zaawansowane zapytania.

