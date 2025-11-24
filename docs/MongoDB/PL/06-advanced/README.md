# 6. Funkcje zaawansowane MongoDB

## 🎯 Cele

- Używać transakcji
- Zrozumieć replikację
- Opanować sharding
- Używać wyszukiwania tekstu
- Funkcje zaawansowane

## 📋 Spis treści

1. [Transakcje](#transakcje)
2. [Replikacja](#replikacja)
3. [Sharding](#sharding)
4. [Wyszukiwanie tekstu](#wyszukiwanie-tekstu)
5. [Inne funkcje](#inne-funkcje)

---

## Transakcje

### Czym jest transakcja?

**Transakcja** = Grupa operacji atomowych

- **Atomowa** : Wszystko lub nic
- **Spójność** : Spójne dane
- **Izolacja** : Izolowane operacje
- **Trwałość** : Trwałe zmiany

### Używać transakcji

```javascript
const session = db.getMongo().startSession()
session.startTransaction()

try {
  db.users.insertOne({name: "John"}, {session})
  db.orders.insertOne({user_id: "...", items: [...]}, {session})
  session.commitTransaction()
} catch (error) {
  session.abortTransaction()
} finally {
  session.endSession()
}
```

---

## Replikacja

### Czym jest replikacja?

**Replikacja** = Wiele kopii danych

- **Wysoka dostępność** : Brak pojedynczego punktu awarii
- **Nadmiarowość** : Automatyczna kopia zapasowa
- **Wydajność** : Odczyt z wielu serwerów

---

## Sharding

### Czym jest sharding?

**Sharding** = Partycjonowanie poziome

- **Skalowalność** : Rozkładać dane
- **Wydajność** : Przetwarzać równolegle
- **Magazyn** : Więcej pojemności

---

## Wyszukiwanie tekstu

### Indeks tekstu

```javascript
// Utworzyć indeks tekstu
db.articles.createIndex({
  title: "text",
  content: "text"
})

// Wyszukiwać
db.articles.find({
  $text: {$search: "mongodb tutorial"}
})
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **Transakcje** : Operacje atomowe
2. **Replikacja** : Wysoka dostępność
3. **Sharding** : Skalowalność pozioma
4. **Wyszukiwanie tekstu** : Wyszukiwanie tekstu
5. **Walidacja** : Opcjonalne schematy

## 🔗 Następny moduł

Przejdź do modułu [7. Dobre praktyki](./07-best-practices/README.md), aby poznać najlepsze praktyki.

