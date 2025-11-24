# 2. Amazon S3 - Przechowywanie danych

## 🎯 Cele

- Zrozumieć Amazon S3 i jego użycie
- Tworzyć i zarządzać bucketami S3
- Przesyłać i organizować pliki
- Zrozumieć klasy przechowywania
- Integrować S3 z innymi usługami AWS

## 📋 Spis treści

1. [Wprowadzenie do S3](#wprowadzenie-do-s3)
2. [Utworzyć bucket S3](#utworzyć-bucket-s3)
3. [Przesyłać i zarządzać plikami](#przesyłać-i-zarządzać-plikami)
4. [Klasy przechowywania](#klasy-przechowywania)
5. [Organizacja danych](#organizacja-danych)
6. [Integracja z innymi usługami](#integracja-z-innymi-usługami)

---

## Wprowadzenie do S3

### Czym jest Amazon S3?

**Amazon S3** (Simple Storage Service) = Usługa przechowywania obiektów

- Nieograniczone przechowywanie
- Wysoka dostępność (99.99%)
- Bezpieczne domyślnie
- Integracja ze wszystkimi usługami AWS

### Przypadki użycia dla Data Analyst

- **Data Lake** : Przechowywać surowe dane
- **Backup** : Tworzyć kopie zapasowe danych
- **ETL** : Źródło/destynacja dla pipeline'ów
- **Analytics** : Dane dla Athena, Redshift
- **Archiwizacja** : Dane historyczne

### Free Tier S3

**Darmowe na zawsze :**
- 5 GB standardowego przechowywania
- 20 000 żądań GET
- 2 000 żądań PUT
- 15 GB transferu danych wychodzących

**⚠️ Ważne :** Poza tymi limitami, normalne rozliczanie.

---

## Utworzyć bucket S3

### Krok 1 : Dostęp do S3

1. Konsola AWS → Szukać "S3"
2. Kliknąć "Amazon S3"
3. Kliknąć "Create bucket"

### Krok 2 : Konfiguracja bucketa

**Podstawowe informacje :**
- **Bucket name** : Nazwa unikalna globalnie (np. `my-data-analyst-bucket`)
- **Region** : Wybrać najbliższy region (np. `eu-west-3` Paryż)

**Opcje konfiguracji :**

1. **Object Ownership**
   - "ACLs disabled" (zalecane)
   - "Bucket owner enforced"

2. **Block Public Access**
   - ✅ **Wszystko włączyć** (bezpieczeństwo domyślne)
   - Wyłączyć tylko w przypadku konkretnej potrzeby

3. **Versioning**
   - Wyłączone domyślnie (darmowe)
   - Włączyć jeśli potrzeba wielu wersji

4. **Tags** (opcjonalne)
   - Dodać tagi do organizacji
   - Np. `Project: Data-Analyst-Training`

5. **Default encryption**
   - ✅ Włączyć (zalecane)
   - "Amazon S3 managed keys (SSE-S3)" (darmowe)

### Krok 3 : Utworzyć bucket

1. Kliknąć "Create bucket"
2. Bucket utworzony i widoczny na liście
3. Gotowy do użycia

**⚠️ Ważne :** Nazwa bucketa musi być unikalna globalnie w AWS.

---

## Przesyłać i zarządzać plikami

### Przesłać plik

**Metoda 1 : Interfejs web**

1. Kliknąć nazwę bucketa
2. Kliknąć "Upload"
3. "Add files" lub "Add folder"
4. Wybrać pliki
5. Kliknąć "Upload"

**Metoda 2 : AWS CLI**

```bash
# Zainstalować AWS CLI (jeśli jeszcze nie)
# Windows: https://aws.amazon.com/cli/
# Linux/Mac: pip install awscli

# Skonfigurować credentials
aws configure

# Przesłać plik
aws s3 cp local-file.csv s3://my-data-analyst-bucket/data/
```

**Metoda 3 : SDK Python (boto3)**

```python
import boto3

# Utworzyć klienta S3
s3 = boto3.client('s3')

# Przesłać plik
s3.upload_file('local-file.csv', 'my-data-analyst-bucket', 'data/file.csv')
```

### Pobrać plik

**Interfejs web :**
1. Kliknąć plik
2. Kliknąć "Download"

**AWS CLI :**
```bash
aws s3 cp s3://my-data-analyst-bucket/data/file.csv local-file.csv
```

**Python :**
```python
s3.download_file('my-data-analyst-bucket', 'data/file.csv', 'local-file.csv')
```

### Zarządzać plikami

**Dostępne akcje :**
- **Download** : Pobierać
- **Open** : Otwierać w przeglądarce
- **Copy** : Kopiować do innej lokalizacji
- **Move** : Przenosić
- **Delete** : Usuwać
- **Make public** : Uczynić publicznym (uwaga bezpieczeństwo)

---

## Klasy przechowywania

### S3 Standard (domyślne)

**Użycie :**
- Dane często dostępne
- Aplikacje produkcyjne

**Charakterystyka :**
- Szybki dostęp
- 99.99% dostępności
- Koszt : ~0.023$ za GB/miesiąc

**Free Tier :** 5 GB darmowe

### S3 Intelligent-Tiering

**Użycie :**
- Dane ze zmiennym dostępem
- Automatyczna optymalizacja kosztów

**Charakterystyka :**
- Automatycznie przenosi między klasami
- Brak opłat za odzyskiwanie
- Koszt : ~0.023$ za GB/miesiąc

### S3 Standard-IA (Infrequent Access)

**Użycie :**
- Dane rzadko dostępne
- Backup, archiwa

**Charakterystyka :**
- Szybki dostęp gdy potrzebny
- Koszt przechowywania : ~0.0125$ za GB/miesiąc
- Koszt odzyskiwania : ~0.01$ za GB

### S3 One Zone-IA

**Użycie :**
- Dane reprodukowalne
- Backup drugorzędny

**Charakterystyka :**
- Przechowywanie w jednej strefie
- Koszt : ~0.01$ za GB/miesiąc
- ⚠️ Ryzyko utraty jeśli strefa ulegnie awarii

### S3 Glacier

**Użycie :**
- Archiwizacja długoterminowa
- Dane rzadko potrzebne

**Charakterystyka :**
- Odzyskiwanie : 1-5 minut do kilku godzin
- Koszt : ~0.004$ za GB/miesiąc
- Opłaty za odzyskiwanie według prędkości

### Wybrać klasę przechowywania

**Dla Data Analyst :**
- **S3 Standard** : Dane aktywne (częste analizy)
- **S3 Standard-IA** : Dane historyczne (okazjonalne analizy)
- **S3 Glacier** : Archiwa (rzadko używane)

**Automatyczne przejście :**
- Skonfigurować reguły przejścia
- Przykład : Standard → Standard-IA po 30 dniach

---

## Organizacja danych

### Zalecana struktura

**Organizacja według projektu :**
```
bucket-name/
├── raw/              # Dane surowe
│   ├── 2024/
│   │   ├── 01/
│   │   ├── 02/
│   │   └── ...
├── processed/        # Dane przekształcone
│   ├── 2024/
│   └── ...
├── analytics/        # Dane do analizy
│   └── ...
└── archive/          # Archiwa
    └── ...
```

**Organizacja według typu :**
```
bucket-name/
├── csv/
├── json/
├── parquet/
└── logs/
```

### Prefiksy i foldery

**S3 nie ma "prawdziwych" folderów**, ale używa prefiksów :

- `data/2024/01/file.csv` = Prefiks `data/2024/01/`
- Interfejs web symuluje foldery
- Używać `/` do organizacji

**Dobre praktyki :**
- Używać spójnych prefiksów
- Uwzględniać datę w ścieżce
- Rozdzielać według typu danych

---

## Integracja z innymi usługami

### S3 + AWS Glue

**Użycie :**
- S3 jako źródło danych
- Glue przekształca dane
- Wynik do S3 lub innej destynacji

**Przykład :**
```python
# Job Glue czyta z S3
datasource = glueContext.create_dynamic_frame.from_catalog(
    database = "my_database",
    table_name = "s3_data"
)
```

### S3 + Amazon Athena

**Użycie :**
- Zapytania SQL bezpośrednio na plikach S3
- Nie potrzeba ładować do bazy danych
- Pay-per-query

**Przykład :**
```sql
-- Utworzyć tabelę zewnętrzną wskazującą na S3
CREATE EXTERNAL TABLE my_table (
    id INT,
    name STRING
)
STORED AS PARQUET
LOCATION 's3://my-bucket/data/';
```

### S3 + Amazon Redshift

**Użycie :**
- S3 jako źródło dla COPY
- Redshift jako data warehouse
- Szybkie ładowanie dużych ilości

**Przykład :**
```sql
COPY my_table
FROM 's3://my-bucket/data/file.csv'
IAM_ROLE 'arn:aws:iam::account:role/RedshiftRole'
CSV;
```

### S3 + AWS Lambda

**Użycie :**
- Wyzwalać Lambda przy przesłaniu
- Automatyczne przetwarzanie plików
- Przekształcanie, walidacja, etc.

**Konfiguracja :**
1. S3 → Properties → Event notifications
2. Utworzyć powiadomienie
3. Wyzwalacz : "All object create events"
4. Destynacja : Funkcja Lambda

---

## Dobre praktyki

### Bezpieczeństwo

1. **Nigdy nie czynić bucketów publicznymi** (chyba że konkretna potrzeba)
2. **Używać IAM** do kontroli dostępu
3. **Włączyć szyfrowanie** domyślnie
4. **Używać bucket policies** dla szczegółowych uprawnień

### Wydajność

1. **Używać prefiksów** do rozłożenia obciążenia
2. **Unikać nazw sekwencyjnych** (np. file1, file2, file3)
3. **Używać Multipart Upload** dla dużych plików (>100MB)
4. **Włączyć Transfer Acceleration** jeśli potrzeba (płatne)

### Koszty

1. **Monitorować użycie** regularnie
2. **Używać odpowiednich klas** przechowywania
3. **Usuwać niepotrzebne pliki**
4. **Konfigurować automatyczne przejścia**
5. **Używać S3 Lifecycle** do automatyzacji

### Organizacja

1. **Nazywać buckety** spójnie
2. **Używać tagów** do organizacji
3. **Dokumentować strukturę** danych
4. **Tworzyć konwencje** nazewnictwa

---

## Przykłady praktyczne

### Przykład 1 : Przesłać plik CSV

```python
import boto3
import pandas as pd

# Utworzyć klienta S3
s3 = boto3.client('s3')

# Czytać plik lokalny
df = pd.read_csv('data.csv')

# Przesłać do S3
s3.upload_file('data.csv', 'my-bucket', 'raw/2024/data.csv')
```

### Przykład 2 : Listować pliki z prefiksu

```python
# Listować wszystkie pliki w prefiksie
response = s3.list_objects_v2(
    Bucket='my-bucket',
    Prefix='raw/2024/'
)

for obj in response.get('Contents', []):
    print(obj['Key'], obj['Size'])
```

### Przykład 3 : Pobrać i przetworzyć

```python
# Pobrać z S3
s3.download_file('my-bucket', 'raw/data.csv', 'local-data.csv')

# Przetworzyć
df = pd.read_csv('local-data.csv')
# ... przetwarzanie ...

# Przesłać wynik
df.to_csv('processed-data.csv', index=False)
s3.upload_file('processed-data.csv', 'my-bucket', 'processed/data.csv')
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **S3 = Nieograniczone przechowywanie** i wysoka dostępność
2. **Free Tier : 5 GB** zawsze darmowe
3. **Organizować z prefiksami** dla lepszej wydajności
4. **Wybrać odpowiednią klasę** według użycia
5. **S3 integruje się** ze wszystkimi usługami danych AWS

## 🔗 Następny moduł

Przejdź do modułu [3. AWS Glue - ETL Serverless](../03-glue/README.md), aby nauczyć się przekształcać dane z AWS Glue.

