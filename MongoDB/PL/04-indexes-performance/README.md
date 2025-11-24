# 4. Indeksy i Wydajność MongoDB

## 🎯 Cele

- Zrozumieć indeksy
- Tworzyć różne typy indeksów
- Analizować wydajność
- Optymalizować zapytania
- Używać explain()

## 📋 Spis treści

1. [Wprowadzenie do indeksów](#wprowadzenie-do-indeksów)
2. [Typy indeksów](#typy-indeksów)
3. [Tworzyć indeksy](#tworzyć-indeksy)
4. [Analizować wydajność](#analizować-wydajność)
5. [Optymalizacja](#optymalizacja)

---

## Wprowadzenie do indeksów

### Czym jest indeks?

**Indeks** = Struktura danych do przyspieszenia zapytań

- **Wydajność** : Szybsze wyszukiwanie
- **Koszt** : Dodatkowa przestrzeń dyskowa
- **Konserwacja** : Automatyczne aktualizacje

---

## Typy indeksów

### Indeks prosty

```javascript
// Indeks na jednym polu
db.users.createIndex({email: 1})
```

### Indeks złożony

```javascript
// Indeks na wielu polach
db.users.createIndex({name: 1, age: -1})
```

### Indeks unikalny

```javascript
// Zapewnić unikalność
db.users.createIndex({email: 1}, {unique: true})
```

---

## Tworzyć indeksy

### Metody tworzenia

```javascript
// Utworzyć indeks
db.collection.createIndex({field: 1})

// Zobaczyć indeksy
db.collection.getIndexes()

// Usunąć indeks
db.collection.dropIndex({field: 1})
```

---

## Analizować wydajność

### explain()

```javascript
// Plan wykonania
db.users.find({email: "john@example.com"}).explain()

// Szczegółowe statystyki
db.users.find({email: "john@example.com"}).explain("executionStats")
```

---

## Optymalizacja

### Dobre praktyki

**1. Indeksować często wyszukiwane pola:**

```javascript
db.users.createIndex({email: 1})
```

**2. Indeks złożony dla wielu zapytań:**

```javascript
db.users.createIndex({name: 1, age: 1})
```

**3. Unikać zbyt wielu indeksów:**

- Każdy indeks spowalnia zapisy
- Używać tylko niezbędnych indeksów

---

## 📊 Kluczowe punkty do zapamiętania

1. **Indeksy** : Przyspieszają wyszukiwania
2. **Typy** : Prosty, złożony, unikalny
3. **explain()** : Analizować wydajność
4. **Optymalizacja** : Indeksować częste pola
5. **Równowaga** : Nie za dużo indeksów

## 🔗 Następny moduł

Przejdź do modułu [5. Modelowanie danych](./05-data-modeling/README.md), aby nauczyć się modelowania.

