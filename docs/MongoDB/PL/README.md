# Szkolenie MongoDB dla Data Analyst

## 📚 Przegląd

To szkolenie poprowadzi Cię przez naukę **MongoDB** jako Data Analyst. MongoDB to baza danych NoSQL zorientowana na dokumenty, idealna do zarządzania danymi nieustrukturyzowanymi i częściowo ustrukturyzowanymi.

## 🎯 Cele szkoleniowe

- Zrozumieć MongoDB i NoSQL
- Zainstalować MongoDB
- Opanować operacje CRUD
- Używać zapytań i agregacji
- Optymalizować z indeksami
- Modelować dane
- Integrować MongoDB w przepływy danych
- Tworzyć praktyczne projekty do portfolio

## 💰 Wszystko jest darmowe!

To szkolenie używa tylko:
- ✅ **MongoDB Community Server** : Darmowy i open-source
- ✅ **MongoDB Compass** : Darmowy interfejs graficzny
- ✅ **MongoDB Atlas** : Darmowy klaster (512 MB)
- ✅ **Oficjalna dokumentacja** : Kompletne darmowe przewodniki
- ✅ **Tutoriale online** : Darmowe zasoby

**Całkowity budżet: 0 zł**

## 📖 Struktura szkolenia

### 1. [Rozpoczęcie z MongoDB](./01-getting-started/README.md)
   - Zainstalować MongoDB
   - Podstawowe koncepcje
   - Pierwsze operacje
   - Interfejs MongoDB Compass

### 2. [Operacje podstawowe](./02-basic-operations/README.md)
   - CRUD (Create, Read, Update, Delete)
   - Kolekcje i Dokumenty
   - Typy danych
   - Operatory zapytań

### 3. [Zapytania i Agregacja](./03-queries-aggregation/README.md)
   - Zaawansowane zapytania
   - Pipeline agregacji
   - Operatory agregacji
   - Grupowanie i obliczenia

### 4. [Indeksy i Wydajność](./04-indexes-performance/README.md)
   - Tworzyć indeksy
   - Typy indeksów
   - Analiza wydajności
   - Optymalizacja zapytań

### 5. [Modelowanie danych](./05-data-modeling/README.md)
   - Modele danych
   - Relacje (Embedded vs References)
   - Elastyczne schematy
   - Dobre praktyki

### 6. [Funkcje zaawansowane](./06-advanced/README.md)
   - Transakcje
   - Replikacja
   - Sharding
   - Wyszukiwanie tekstu

### 7. [Dobre praktyki](./07-best-practices/README.md)
   - Bezpieczeństwo
   - Wydajność
   - Konserwacja
   - Backup i Restore

### 8. [Projekty praktyczne](./08-projets/README.md)
   - Aplikacja Python z MongoDB
   - Pipeline danych
   - Analiza danych
   - Projekty do portfolio

## 🚀 Szybki start

### Wymagania wstępne

- **System operacyjny** : Windows, Linux lub macOS
- **4 GB RAM** : Minimum zalecane
- **Miejsce na dysku** : 5 GB wolne

### Szybka instalacja

**Windows:**
1. Pobrać MongoDB: https://www.mongodb.com/try/download/community
2. Zainstalować z opcjami domyślnymi
3. Sprawdzić: `mongod --version`

**Linux:**
```bash
wget -qO - https://www.mongodb.org/static/pgp/server-7.0.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
sudo apt-get update
sudo apt-get install -y mongodb-org
sudo systemctl start mongod
```

**macOS:**
```bash
brew tap mongodb/brew
brew install mongodb-community
brew services start mongodb-community
```

### Pierwszy test

```bash
mongod
mongosh
use test
db.collection.insertOne({name: "test"})
db.collection.find()
```

## 📊 Przypadki użycia dla Data Analyst

- **Dane nieustrukturyzowane** : JSON, logi, API
- **Elastyczność** : Ewoluujące schematy
- **Agregacja** : Potężny pipeline do analizy
- **Integracja** : Z Python, R, PowerBI
- **Big Data** : Skalowalność pozioma

## 📚 Darmowe zasoby

### Oficjalna dokumentacja

- **Dokumentacja MongoDB** : https://docs.mongodb.com/
- **MongoDB University** : https://university.mongodb.com/
- **MongoDB Compass** : https://www.mongodb.com/products/compass

