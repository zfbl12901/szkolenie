# 3. Zapytania i Agregacja MongoDB

## 🎯 Cele

- Opanować zaawansowane zapytania
- Używać pipeline agregacji
- Stosować operatory agregacji
- Wykonywać grupowanie i obliczenia
- Analizować złożone dane

## 📋 Spis treści

1. [Zaawansowane zapytania](#zaawansowane-zapytania)
2. [Pipeline agregacji](#pipeline-agregacji)
3. [Operatory agregacji](#operatory-agregacji)
4. [Grupowanie i obliczenia](#grupowanie-i-obliczenia)
5. [Przykłady praktyczne](#przykłady-praktyczne)

---

## Zaawansowane zapytania

### Projekcja

```javascript
// Wybrać konkretne pola
db.users.find({}, {name: 1, email: 1, _id: 0})
```

### Sortowanie i limit

```javascript
// Sortować według wieku (rosnąco)
db.users.find().sort({age: 1})

// Ograniczyć wyniki
db.users.find().limit(10)
```

---

## Pipeline agregacji

### Czym jest Pipeline?

**Pipeline** = Seria kroków transformacji

- **Kroki** : Każdy krok przekształca dane
- **Sekwencyjny** : Wynik jednego kroku = wejście następnego
- **Potężny** : Do złożonej analizy

### Podstawowa struktura

```javascript
db.collection.aggregate([
  { $match: { ... } },      // Filtrować
  { $group: { ... } },       // Grupować
  { $sort: { ... } },        // Sortować
  { $project: { ... } }      // Wybierać
])
```

---

## Operatory agregacji

### $match

```javascript
db.sales.aggregate([
  {$match: {amount: {$gt: 500}}}
])
```

### $group

```javascript
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

---

## Grupowanie i obliczenia

### Operatory akumulacji

```javascript
// Suma
{$sum: "$amount"}

// Średnia
{$avg: "$price"}

// Minimum
{$min: "$price"}

// Maximum
{$max: "$price"}

// Liczenie
{$sum: 1}
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **Pipeline** : Seria kroków transformacji
2. **$match** : Filtrować dokumenty
3. **$group** : Grupować i obliczać
4. **$project** : Wybierać i przekształcać
5. **Agregacja** : Potężna do analizy

## 🔗 Następny moduł

Przejdź do modułu [4. Indeksy i Wydajność](./04-indexes-performance/README.md), aby optymalizować wydajność.

