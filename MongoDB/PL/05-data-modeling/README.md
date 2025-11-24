# 5. Modelowanie danych MongoDB

## 🎯 Cele

- Zrozumieć modele danych
- Wybierać między Embedded i References
- Projektować elastyczne schematy
- Stosować dobre praktyki
- Optymalizować strukturę

## 📋 Spis treści

1. [Modele danych](#modele-danych)
2. [Embedded vs References](#embedded-vs-references)
3. [Elastyczne schematy](#elastyczne-schematy)
4. [Dobre praktyki](#dobre-praktyki)
5. [Przykłady praktyczne](#przykłady-praktyczne)

---

## Modele danych

### Model Embedded (Zagnieżdżony)

**Wszystko w jednym dokumencie:**

```javascript
{
  _id: ObjectId("..."),
  name: "John",
  address: {
    street: "123 Main St",
    city: "Warsaw"
  }
}
```

**Zalety:**
- Szybki dostęp (jeden dokument)
- Brak joinów
- Spójne dane

### Model References (Referencje)

**Osobne dokumenty z referencjami:**

```javascript
// Kolekcja users
{_id: ObjectId("..."), name: "John"}

// Kolekcja addresses
{_id: ObjectId("..."), user_id: ObjectId("..."), street: "123 Main St"}
```

**Zalety:**
- Brak limitu rozmiaru
- Brak duplikacji
- Elastyczność

---

## Embedded vs References

### Kiedy używać Embedded?

**Przypadki użycia:**
- Dane często dostępne razem
- Małe ilości danych
- Relacja 1:1 lub 1:kilka
- Dane rzadko się zmieniają

### Kiedy używać References?

**Przypadki użycia:**
- Duże ilości danych
- Relacja 1:wiele lub wiele:wiele
- Dane współdzielone
- Dane często się zmieniają

---

## Elastyczne schematy

### Ewolucja schematu

```javascript
// Dokument początkowy
{name: "John", age: 30}

// Dodać pole później
{name: "John", age: 30, email: "john@example.com"}
```

---

## Dobre praktyki

### Wzorce modelowania

**One-to-Few:**
```javascript
// Embedded
{name: "John", addresses: [{street: "123 Main St"}]}
```

**One-to-Many:**
```javascript
// References
// Kolekcja users
{_id: ObjectId("..."), name: "John"}

// Kolekcja orders
{user_id: ObjectId("..."), items: [...]}
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **Embedded** : Dla danych często dostępnych razem
2. **References** : Dla dużych ilości lub złożonych relacji
3. **Elastyczność** : Ewoluujące schematy
4. **Wzorce** : One-to-Few, One-to-Many, Many-to-Many
5. **Wydajność** : Równoważyć dostęp i spójność

## 🔗 Następny moduł

Przejdź do modułu [6. Funkcje zaawansowane](./06-advanced/README.md), aby pogłębić.

