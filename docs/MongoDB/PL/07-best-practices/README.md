# 7. Dobre praktyki MongoDB

## 🎯 Cele

- Bezpieczeństwo
- Wydajność
- Konserwacja
- Backup i Restore
- Monitorowanie

## 📋 Spis treści

1. [Bezpieczeństwo](#bezpieczeństwo)
2. [Wydajność](#wydajność)
3. [Konserwacja](#konserwacja)
4. [Backup i Restore](#backup-i-restore)
5. [Monitorowanie](#monitorowanie)

---

## Bezpieczeństwo

### Uwierzytelnianie

```javascript
// Utworzyć użytkownika admin
use admin
db.createUser({
  user: "admin",
  pwd: "secure_password",
  roles: ["root"]
})
```

### Bezpieczne połączenie

```bash
mongosh -u admin -p secure_password --authenticationDatabase admin
```

---

## Wydajność

### Indeksy

```javascript
// Indeksować często wyszukiwane pola
db.users.createIndex({email: 1})

// Indeks złożony dla wielu zapytań
db.orders.createIndex({customer: 1, date: -1})
```

### Zapytania

```javascript
// Używać projekcji do ograniczenia danych
db.users.find({}, {name: 1, email: 1})

// Ograniczać wyniki
db.users.find().limit(100)
```

---

## Konserwacja

### Czyszczenie

```javascript
// Usunąć przestarzałe dokumenty
db.logs.deleteMany({
  created_at: {$lt: new Date("2024-01-01")}
})
```

---

## Backup i Restore

### Backup (mongodump)

```bash
# Backup bazy danych
mongodump --db mydb --out /backup/
```

### Restore (mongorestore)

```bash
# Przywrócić bazę danych
mongorestore --db mydb /backup/mydb/
```

---

## Monitorowanie

### Status serwera

```javascript
// Status serwera
db.serverStatus()

// Bieżące operacje
db.currentOp()
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **Bezpieczeństwo** : Uwierzytelnianie i autoryzacja
2. **Wydajność** : Indeksy i zoptymalizowane zapytania
3. **Konserwacja** : Regularne czyszczenie
4. **Backup** : Regularne kopie zapasowe
5. **Monitorowanie** : Monitorować wydajność

## 🔗 Następny moduł

Przejdź do modułu [8. Projekty praktyczne](./08-projets/README.md), aby tworzyć kompletne projekty.

