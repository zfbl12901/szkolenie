# 3. Azure Data Factory - ETL w chmurze

## 🎯 Cele

- Zrozumieć Azure Data Factory i jego rolę
- Tworzyć pipeline'y ETL
- Używać działań przekształcania
- Integrować ze źródłami danych
- Orkiestrować przepływy pracy

## 📋 Spis treści

1. [Wprowadzenie do Data Factory](#wprowadzenie-do-data-factory)
2. [Utworzyć Data Factory](#utworzyć-data-factory)
3. [Utworzyć pipeline](#utworzyć-pipeline)
4. [Działania przekształcania](#działania-przekształcania)
5. [Integracja ze źródłami danych](#integracja-ze-źródłami-danych)
6. [Orkiestracja i harmonogramowanie](#orkiestracja-i-harmonogramowanie)

---

## Wprowadzenie do Data Factory

### Czym jest Azure Data Factory?

**Azure Data Factory** = Zarządzana usługa ETL w chmurze

- **ETL** : Extract, Transform, Load
- **Chmura** : Brak infrastruktury do zarządzania
- **Zarządzane** : Microsoft zarządza infrastrukturą
- **Skalowalne** : Automatycznie dostosowuje się

### Komponenty Data Factory

1. **Pipelines** : Przepływy pracy ETL
2. **Activities** : Kroki w pipeline
3. **Datasets** : Reprezentacje danych
4. **Linked Services** : Połączenia ze źródłami
5. **Triggers** : Automatyczne wyzwalanie

### Data Factory Free Tier

**Darmowe na zawsze :**
- 5 darmowych pipeline'ów
- Ograniczone działania
- Poza tym : rozliczanie według użycia

**⚠️ Ważne :** Monitorować koszty, zwłaszcza dla działań przekształcania.

---

## Utworzyć Data Factory

### Krok 1 : Dostęp do Data Factory

1. Portal Azure → Szukać "Data Factory"
2. Kliknąć "Data factories"
3. Kliknąć "Create"

### Krok 2 : Podstawowa konfiguracja

**Podstawowe informacje :**
- **Subscription** : Wybrać subskrypcję
- **Resource group** : Utworzyć lub użyć istniejącego
- **Name** : `my-data-factory` (unikalne globalnie)
- **Version** : V2 (zalecane)
- **Region** : Wybrać najbliższy region

**Konfiguracja Git (opcjonalne) :**
- **Configure Git later** : Aby szybko rozpocząć
- Lub skonfigurować Git/GitHub do wersjonowania

### Krok 3 : Utworzyć Data Factory

1. Kliknąć "Review + create"
2. Sprawdzić konfigurację
3. Kliknąć "Create"
4. Czekać na utworzenie (2-3 minuty)

**⚠️ Ważne :** Zanotować nazwę Data Factory.

### Krok 4 : Otworzyć Data Factory Studio

1. Po utworzeniu, kliknąć "Open Azure Data Factory Studio"
2. Interfejs web do tworzenia pipeline'ów

---

## Utworzyć pipeline

### Krok 1 : Utworzyć Linked Service

**Linked Service = Połączenie ze źródłem danych**

**Przykład : Azure Blob Storage**

1. Data Factory Studio → "Manage" → "Linked services"
2. Kliknąć "+ New"
3. Szukać "Azure Blob Storage"
4. Konfiguracja :
   - **Name** : `AzureBlobStorage1`
   - **Storage account name** : Wybrać konto
   - **Authentication method** : Account key (lub inny)
5. Kliknąć "Create"

### Krok 2 : Utworzyć Dataset

**Dataset = Reprezentacja danych**

1. Data Factory Studio → "Author" → "Datasets"
2. Kliknąć "+ New"
3. Wybrać "Azure Blob Storage"
4. Konfiguracja :
   - **Name** : `CSVData`
   - **Linked service** : `AzureBlobStorage1`
   - **File path** : `raw-data/`
   - **File format** : DelimitedText (CSV)
5. Kliknąć "Create"

### Krok 3 : Utworzyć pipeline

1. Data Factory Studio → "Author" → "Pipelines"
2. Kliknąć "+ New pipeline"
3. Nazwać pipeline : `CopyCSVToParquet`

### Krok 4 : Dodać działanie

**Przykład : Copy Data**

1. W pipeline, przeciągnąć "Copy Data" z "Move & transform"
2. Skonfigurować :
   - **Source** : Dataset `CSVData`
   - **Sink (Destination)** : Utworzyć nowy dataset Parquet
3. Kliknąć "Publish" aby zapisać

---

## Działania przekształcania

### Copy Data

**Kopiować dane ze źródła do miejsca docelowego**

**Konfiguracja :**
- **Source** : Dataset źródłowy
- **Sink** : Dataset docelowy
- **Mapping** : Mapowanie kolumn

**Przykład : CSV → Parquet**

```json
{
  "name": "CopyCSVToParquet",
  "type": "Copy",
  "inputs": [{"referenceName": "CSVData"}],
  "outputs": [{"referenceName": "ParquetData"}],
  "typeProperties": {
    "source": {"type": "DelimitedTextSource"},
    "sink": {"type": "ParquetSink"}
  }
}
```

### Data Flow

**Przekształcanie danych z interfejsem graficznym**

**Kroki :**
1. Utworzyć Data Flow
2. Dodać źródło
3. Dodać przekształcenia :
   - **Select** : Wybierać kolumny
   - **Filter** : Filtrować wiersze
   - **Derived Column** : Tworzyć kolumny obliczane
   - **Aggregate** : Agregacje
   - **Join** : Łączyć dane
4. Dodać sink

**Przykład przekształceń :**

```
Source (CSV) 
  → Select (kolumny)
  → Filter (status = 'active')
  → Derived Column (nowa kolumna)
  → Aggregate (SUM, COUNT)
  → Sink (Parquet)
```

### Lookup

**Wyszukiwać wartości w innym źródle**

**Użycie :**
- Walidować dane
- Wzbogacać dane
- Sprawdzać referencje

### Stored Procedure

**Wykonać procedurę składowaną SQL**

**Użycie :**
- Przetwarzanie w SQL Database
- Złożona logika biznesowa
- Optymalizacja po stronie bazy

---

## Integracja ze źródłami danych

### Azure Blob Storage

**Źródło danych :**

```json
{
  "type": "AzureBlobStorage",
  "typeProperties": {
    "connectionString": "...",
    "container": "raw-data"
  }
}
```

### Azure SQL Database

**Źródło danych :**

```json
{
  "type": "AzureSqlDatabase",
  "typeProperties": {
    "connectionString": "...",
    "tableName": "users"
  }
}
```

### Azure Data Lake Storage Gen2

**Źródło danych :**

```json
{
  "type": "AzureBlobFS",
  "typeProperties": {
    "url": "https://account.dfs.core.windows.net",
    "fileSystem": "data-lake"
  }
}
```

### Pliki lokalne (przez Self-hosted IR)

**Integration Runtime :**
- Self-hosted IR do dostępu do plików lokalnych
- Zainstalować na maszynie lokalnej
- Połączyć z Data Factory

---

## Orkiestracja i harmonogramowanie

### Wyzwalać ręcznie

1. Data Factory Studio → "Monitor"
2. Wybrać pipeline
3. Kliknąć "Trigger now"
4. Zobaczyć wykonanie w czasie rzeczywistym

### Planować pipeline (Trigger)

**Utworzyć trigger :**

1. Pipeline → "Add trigger" → "New/Edit"
2. Typ : "Schedule"
3. Konfiguracja :
   - **Name** : `DailyTrigger`
   - **Type** : Schedule
   - **Recurrence** : Daily
   - **Start time** : 02:00
4. Kliknąć "OK"

**Typy triggerów :**
- **Schedule** : Planowane (cron)
- **Event** : Wyzwalane przez zdarzenie
- **Tumbling window** : Okno przesuwne

### Wyzwalać przez zdarzenie

**Przykład : Nowy plik w Blob Storage**

1. Utworzyć trigger "Storage event"
2. Skonfigurować :
   - **Storage account** : Twoje konto
   - **Container** : `raw-data`
   - **Event type** : Blob created
3. Powiązać z pipeline

---

## Dobre praktyki

### Wydajność

1. **Używać Data Flow** dla złożonych przekształceń
2. **Optymalizować działania** aby zmniejszyć czas
3. **Używać równoległości** gdy możliwe
4. **Wybrać odpowiednie regiony** aby zmniejszyć opóźnienie

### Koszty

1. **Monitorować wykonania** w Monitor
2. **Używać 5 darmowych pipeline'ów** mądrze
3. **Optymalizować Data Flows** (kosztowne)
4. **Zatrzymywać nieużywane pipeline'y**

### Organizacja

1. **Nazywać jasno** pipeline'y i działania
2. **Dokumentować** przekształcenia
3. **Wersjonować** z Git
4. **Testować** przed publikacją

### Bezpieczeństwo

1. **Używać Key Vault** dla sekretów
2. **Ograniczać uprawnienia** Linked Services
3. **Audytować** wykonania
4. **Szyfrować** dane w tranzycie

---

## Przykłady praktyczne

### Przykład 1 : Prosty pipeline CSV → Parquet

**Pipeline :**
1. Źródło : Azure Blob Storage (CSV)
2. Działanie : Copy Data
3. Sink : Azure Blob Storage (Parquet)

**Konfiguracja :**
- Źródło : `raw-data/data.csv`
- Sink : `processed-data/data.parquet`
- Format : DelimitedText → Parquet

### Przykład 2 : Pipeline z przekształceniem

**Pipeline :**
1. Źródło : Azure SQL Database
2. Data Flow :
   - Wybrać kolumny
   - Filtrować wiersze
   - Agregować
3. Sink : Azure Blob Storage (Parquet)

### Przykład 3 : Pipeline orkiestrowany

**Pipeline :**
1. Lookup : Sprawdzić czy nowe dane
2. If Condition : Jeśli nowe dane
3. Copy Data : Skopiować do staging
4. Data Flow : Przekształcić
5. Copy Data : Załadować do miejsca docelowego

---

## 📊 Kluczowe punkty do zapamiętania

1. **Data Factory = ETL w chmurze** zarządzane przez Microsoft
2. **Free Tier : 5 pipeline'ów** darmowych
3. **Pipelines** orkiestrują działania
4. **Data Flows** dla złożonych przekształceń
5. **Triggers** umożliwiają automatyzację

## 🔗 Następny moduł

Przejdź do modułu [4. Azure SQL Database - Baza danych](../04-sql-database/README.md), aby nauczyć się używać SQL Database na Azure.

