# 1. Rozpoczęcie z MongoDB

## 🎯 Cele

- Zrozumieć MongoDB i NoSQL
- Zainstalować MongoDB
- Zrozumieć podstawowe koncepcje
- Używać MongoDB Compass
- Pierwsze operacje

## 📋 Spis treści

1. [Wprowadzenie do MongoDB](#wprowadzenie-do-mongodb)
2. [Instalacja](#instalacja)
3. [Podstawowe koncepcje](#podstawowe-koncepcje)
4. [MongoDB Compass](#mongodb-compass)
5. [Pierwsze operacje](#pierwsze-operacje)

---

## Wprowadzenie do MongoDB

### Czym jest MongoDB?

**MongoDB** = Baza danych NoSQL zorientowana na dokumenty

- **NoSQL** : Nierelacyjna
- **Dokumenty** : Przechowywanie w formacie JSON (BSON)
- **Elastyczna** : Ewoluujące schematy
- **Skalowalna** : Skalowalność pozioma
- **Open-source** : Darmowa i open-source

### Dlaczego MongoDB dla Data Analyst?

- **Dane nieustrukturyzowane** : JSON, logi, API
- **Elastyczność** : Ewoluujące schematy
- **Agregacja** : Potężny pipeline do analizy
- **Integracja** : Z Python, R, PowerBI
- **Wydajność** : Szybka dla złożonych zapytań

---

## Instalacja

### Windows

1. Przejść do: https://www.mongodb.com/try/download/community
2. Wybrać Windows
3. Pobrać instalator MSI
4. Uruchomić instalator
5. Wybrać instalację "Complete"

### Linux

**Ubuntu/Debian:**
```bash
wget -qO - https://www.mongodb.org/static/pgp/server-7.0.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
sudo apt-get update
sudo apt-get install -y mongodb-org
sudo systemctl start mongod
```

### macOS

**Z Homebrew:**
```bash
brew tap mongodb/brew
brew install mongodb-community
brew services start mongodb-community
```

---

## Podstawowe koncepcje

### Baza danych

**Baza danych** = Kontener kolekcji

- **Auto-tworzenie** : Tworzona przy pierwszym użyciu
- **Nazwa** : Unikalny identyfikator
- **Kolekcje** : Zawiera kolekcje

### Kolekcja

**Kolekcja** = Grupa dokumentów

- **Równoważna** : Tabela w SQL
- **Elastyczna** : Brak narzuconego schematu
- **Dokumenty** : Zawiera dokumenty

### Dokument

**Dokument** = Rekord w formacie JSON

- **Format** : BSON (Binary JSON)
- **Elastyczny** : Zmienna struktura
- **Pola** : Pary klucz-wartość

---

## MongoDB Compass

### Czym jest Compass?

**MongoDB Compass** = Interfejs graficzny

- **Wizualizacja** : Widzieć dane
- **Zapytania** : Wykonywać zapytania
- **Analiza** : Analizować wydajność
- **Zarządzanie** : Zarządzać indeksami

---

## Pierwsze operacje

### Połączyć z mongosh

```bash
# Uruchomić mongosh
mongosh

# Zobaczyć bazy danych
show dbs

# Używać bazy danych
use mydb

# Zobaczyć kolekcje
show collections

# Wstawić dokument
db.users.insertOne({name: "John", age: 30, city: "Warsaw"})

# Znaleźć dokumenty
db.users.find()
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **MongoDB** = Baza danych NoSQL zorientowana na dokumenty
2. **Dokumenty** = Format JSON (BSON)
3. **Kolekcje** = Grupy dokumentów
4. **Bazy danych** = Kontenery kolekcji
5. **Compass** = Interfejs graficzny

## 🔗 Następny moduł

Przejdź do modułu [2. Operacje podstawowe](./02-basic-operations/README.md), aby opanować CRUD.

