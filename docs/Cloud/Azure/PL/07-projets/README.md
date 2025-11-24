# 7. Projekty praktyczne Azure

## 🎯 Cele

- Stosować zdobytą wiedzę
- Tworzyć kompletne pipeline'y ETL
- Integrować z PowerBI
- Tworzyć projekty dla portfolio
- Używać wielu usług Azure

## 📋 Spis treści

1. [Projekt 1 : Pipeline ETL Blob → SQL Database](#projekt-1--pipeline-etl-blob---sql-database)
2. [Projekt 2 : Data Lake z Synapse](#projekt-2--data-lake-z-synapse)
3. [Projekt 3 : Analytics z PowerBI](#projekt-3--analytics-z-powerbi)
4. [Projekt 4 : Kompletny zautomatyzowany pipeline](#projekt-4--kompletny-zautomatyzowany-pipeline)
5. [Dobre praktyki dla portfolio](#dobre-praktyki-dla-portfolio)

---

## Projekt 1 : Pipeline ETL Blob → SQL Database

### Cel

Utworzyć pipeline ETL który ładuje pliki CSV z Blob Storage do SQL Database.

### Architektura

```
Blob Storage (CSV) → Data Factory → SQL Database → PowerBI
```

### Kroki

#### 1. Przygotować dane

**Utworzyć kontener Blob Storage :**
- Nazwa : `raw-data`
- Przesłać plik CSV testowy

**Przykład danych CSV :**
```csv
id,name,email,created_at,status
1,John Doe,john@example.com,2024-01-01,active
2,Jane Smith,jane@example.com,2024-01-02,inactive
```

#### 2. Utworzyć bazę SQL Database

1. Portal Azure → Utworzyć SQL Database
2. Konfiguracja :
   - Name : `analytics-db`
   - Server : Utworzyć nowy serwer
   - Service tier : Basic (darmowe 12 miesięcy)
3. Utworzyć bazę

#### 3. Utworzyć tabelę w SQL Database

```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    created_at DATETIME2,
    status VARCHAR(20)
);
```

#### 4. Utworzyć pipeline Data Factory

1. Data Factory Studio → "Author" → "Pipelines"
2. Utworzyć nowy pipeline : `LoadCSVToSQL`
3. Dodać działanie "Copy Data"
4. Konfiguracja :
   - **Source** : Azure Blob Storage (CSV)
   - **Sink** : Azure SQL Database (tabela users)
5. Opublikować pipeline

#### 5. Wykonać pipeline

1. Kliknąć "Trigger now"
2. Sprawdzić wykonanie w "Monitor"
3. Sprawdzić dane w SQL Database

### Wynik

- Pliki CSV załadowane w SQL Database
- Funkcjonalny pipeline ETL
- Gotowe do analytics z PowerBI

---

## Projekt 2 : Data Lake z Synapse

### Cel

Utworzyć kompletny Data Lake z ingerencją, przekształcaniem i analytics.

### Architektura

```
Źródła → Data Lake Storage (Raw) → Synapse (Transform) → Data Lake (Processed) → PowerBI
                ↓
        Data Factory (Orkiestracja)
```

### Kroki

#### 1. Struktura Data Lake Storage

```
data-lake/
├── raw/
│   ├── users/
│   ├── orders/
│   └── products/
├── processed/
│   ├── users/
│   ├── orders/
│   └── products/
└── analytics/
    └── results/
```

#### 2. Utworzyć obszar roboczy Synapse

1. Portal Azure → Utworzyć Azure Synapse Analytics
2. Konfiguracja :
   - Workspace name : `my-synapse-workspace`
   - Data Lake Storage : Utworzyć nowy
3. Utworzyć obszar roboczy

#### 3. Pipeline'y Data Factory do przekształcania

**Pipeline dla users :**

1. Synapse Studio → "Integrate" → "Pipelines"
2. Utworzyć pipeline : `TransformUsers`
3. Działania :
   - Source : Data Lake Storage (raw/users/)
   - Data Flow : Przekształcić (filtrować, czyścić)
   - Sink : Data Lake Storage (processed/users/)
4. Opublikować

#### 4. Tabele Synapse do analytics

```sql
-- Utworzyć tabelę zewnętrzną
CREATE EXTERNAL TABLE users_processed (
    id INT,
    name VARCHAR(100),
    email VARCHAR(100),
    created_at DATETIME2
)
WITH (
    LOCATION = 'processed/users/',
    DATA_SOURCE = DataLakeStorage,
    FILE_FORMAT = ParquetFormat
);

-- Zapytanie analityczne
SELECT 
    YEAR(created_at) AS year,
    COUNT(*) AS user_count
FROM users_processed
GROUP BY YEAR(created_at);
```

#### 5. Automatyzacja z Triggers

1. Pipeline → "Add trigger" → "New/Edit"
2. Typ : Schedule
3. Recurrence : Daily
4. Start time : 02:00
5. Zapisać

### Wynik

- Funkcjonalny Data Lake
- Zautomatyzowany pipeline
- Analytics z Synapse
- Kompletny projekt dla portfolio

---

## Projekt 3 : Analytics z PowerBI

### Cel

Utworzyć kompletny system analytics z PowerBI połączonym z Azure.

### Kroki

#### 1. Przygotować dane

**W SQL Database lub Synapse :**
- Ładować dane
- Tworzyć widoki analityczne

#### 2. Połączyć PowerBI z Azure SQL Database

1. PowerBI Desktop → "Get Data"
2. "Azure" → "Azure SQL Database"
3. Konfiguracja :
   - Server : `my-sql-server.database.windows.net`
   - Database : `analytics-db`
   - Authentication : Database
4. Wybrać tabele lub widoki
5. Kliknąć "Load"

#### 3. Tworzyć wizualizacje

**Przykład :**
1. Importować tabelę `users`
2. Utworzyć wykres : Liczba użytkowników na miesiąc
3. Dodać filtry
4. Utworzyć dashboard

#### 4. Opublikować na PowerBI Service

1. PowerBI Desktop → "Publish"
2. Wybrać obszar roboczy
3. Opublikować
4. Dostęp do raportu na powerbi.com

#### 5. Odświeżać dane

1. PowerBI Service → Dataset → "Schedule refresh"
2. Konfiguracja :
   - Frequency : Daily
   - Time : 03:00
3. Zapisać

### Wynik

- Analytics z PowerBI
- Interaktywne wizualizacje
- Automatyczne odświeżanie
- Kompletny projekt dla portfolio

---

## Projekt 4 : Kompletny zautomatyzowany pipeline

### Cel

Utworzyć kompletnie zautomatyzowany pipeline ETL z wieloma usługami Azure.

### Kompletna architektura

```
Plik CSV przesłany → Blob Storage (raw/)
    ↓ (Event)
Azure Function (Walidacja)
    ↓
Blob Storage (validated/)
    ↓ (Trigger)
Data Factory Pipeline (Przekształć CSV → Parquet)
    ↓
Data Lake Storage (processed/)
    ↓
Synapse (Analytics)
    ↓
SQL Database (Results)
    ↓
PowerBI (Wizualizacja)
```

### Implementacja

#### 1. Azure Function walidacji

```python
import azure.functions as func
import logging
import csv
from azure.storage.blob import BlobServiceClient

def main(blob: func.InputStream):
    logging.info(f'Processing blob: {blob.name}')
    
    # Czytać blob
    content = blob.read().decode('utf-8')
    reader = csv.DictReader(content.splitlines())
    
    # Walidować
    valid_rows = []
    for row in reader:
        if row.get('email') and '@' in row['email']:
            valid_rows.append(row)
    
    # Przesłać zwalidowane dane
    if valid_rows:
        # Przesłać do validated/
        # ...
    
    logging.info(f'Validated {len(valid_rows)} rows')
```

#### 2. Data Factory Pipeline przekształcania

**Pipeline :**
1. Source : Blob Storage (validated/)
2. Data Flow : Przekształcić (czyścić, wzbogacać)
3. Sink : Data Lake Storage (processed/parquet/)

#### 3. Synapse do analytics

```sql
-- Utworzyć widok analityczny
CREATE VIEW vw_user_analytics AS
SELECT 
    u.id,
    u.name,
    COUNT(o.id) AS order_count,
    SUM(o.amount) AS total_spent
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name;
```

#### 4. PowerBI do wizualizacji

1. Połączyć PowerBI z Synapse
2. Używać widoku `vw_user_analytics`
3. Tworzyć wizualizacje
4. Opublikować raport

### Wynik

- Kompletnie zautomatyzowany pipeline
- Automatyczna walidacja
- Automatyczne przekształcanie
- Analytics dostępne natychmiast
- Wizualizacje PowerBI

---

## Dobre praktyki dla portfolio

### Dokumentacja

**Utworzyć README dla każdego projektu :**

```markdown
# Projekt : Pipeline ETL Azure

## Opis
Zautomatyzowany pipeline ETL do przekształcania danych CSV w Parquet.

## Architektura
- Blob Storage : Przechowywanie
- Data Factory : Przekształcanie
- SQL Database : Baza danych
- PowerBI : Wizualizacja

## Wyniki
- Redukcja kosztów o 50%
- Czas przetwarzania zmniejszony o 70%
```

### Wizualizacje

**Tworzyć diagramy :**
- Architektura systemu
- Przepływ danych
- Schemat danych

**Narzędzia :**
- Draw.io
- Lucidchart
- Diagramy ASCII w README

### Metryki

**Uwzględniać metryki :**
- Czas wykonania przed/po
- Koszty przed/po
- Wolumen przetworzonych danych
- Wydajność zapytań

### Kod

**Dobre praktyki :**
- Kod skomentowany
- Zmienne środowiskowe do konfiguracji
- Obsługa błędów
- Logowanie

### GitHub

**Utworzyć repozytorium :**
- README z dokumentacją
- Skrypty Data Factory (JSON)
- Skrypty SQL
- Konfiguracja
- Diagramy

---

## 📊 Kluczowe punkty do zapamiętania

1. **Projekty praktyczne** : Niezbędne dla portfolio
2. **Dokumentacja** : Wyjaśniać architekturę i wyniki
3. **Metryki** : Pokazywać wpływ (wydajność, koszty)
4. **Czysty kod** : Skomentowany i zorganizowany
5. **GitHub** : Dzielić się projektami

## 🔗 Zasoby

- [Azure Architecture Center](https://learn.microsoft.com/azure/architecture/)
- [Azure Solutions](https://azure.microsoft.com/solutions/)
- [GitHub Azure Examples](https://github.com/Azure-Samples)

---

**Gratulacje !** Ukończyłeś formację Azure dla Data Analyst. Możesz teraz tworzyć kompletne projekty na Azure używając dostępnych darmowych zasobów.

