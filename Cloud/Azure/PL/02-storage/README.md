# 2. Azure Storage - Przechowywanie danych

## 🎯 Cele

- Zrozumieć Azure Storage i jego użycie
- Tworzyć konta przechowywania
- Używać Blob Storage i Data Lake Storage
- Przesyłać i zarządzać plikami
- Organizować dane

## 📋 Spis treści

1. [Wprowadzenie do Azure Storage](#wprowadzenie-do-azure-storage)
2. [Utworzyć konto przechowywania](#utworzyć-konto-przechowywania)
3. [Blob Storage](#blob-storage)
4. [Data Lake Storage Gen2](#data-lake-storage-gen2)
5. [Przesyłać i zarządzać plikami](#przesyłać-i-zarządzać-plikami)
6. [Integracja z innymi usługami](#integracja-z-innymi-usługami)

---

## Wprowadzenie do Azure Storage

### Czym jest Azure Storage?

**Azure Storage** = Zarządzana usługa przechowywania w chmurze

- **Nieograniczone przechowywanie** : Skalowalne według potrzeb
- **Wysoka dostępność** : 99.99% dostępności
- **Bezpieczne** : Szyfrowanie domyślnie
- **Integracja** : Ze wszystkimi usługami Azure

### Typy przechowywania

1. **Blob Storage** : Pliki (CSV, JSON, Parquet, itp.)
2. **Data Lake Storage Gen2** : Data Lake z hierarchicznym systemem plików
3. **File Storage** : Udziały plików
4. **Queue Storage** : Kolejki
5. **Table Storage** : Przechowywanie NoSQL

### Azure Storage Free Tier

**Darmowe 12 miesięcy :**
- 5 GB przechowywania Blob
- 5 GB przechowywania File
- 5 GB przechowywania Table
- 5 GB przechowywania Queue

**Darmowe na zawsze :**
- 200 GB transferu danych wychodzących/miesiąc

**⚠️ Ważne :** Poza tymi limitami, normalne rozliczanie.

---

## Utworzyć konto przechowywania

### Krok 1 : Dostęp do Azure Storage

1. Portal Azure → Szukać "Storage accounts"
2. Kliknąć "Storage accounts"
3. Kliknąć "Create"

### Krok 2 : Podstawowa konfiguracja

**Podstawowe informacje :**
- **Subscription** : Wybrać subskrypcję
- **Resource group** : Utworzyć lub użyć istniejącego
- **Storage account name** : Nazwa unikalna globalnie (np. `mydataanalyststorage`)
- **Region** : Wybrać najbliższy region (np. `France Central`)

**Opcje wydajności :**
- **Performance** : Standard (zalecane do rozpoczęcia)
- **Redundancy** : LRS (Locally Redundant Storage) - najtańsze

### Krok 3 : Opcje zaawansowane

**Bezpieczeństwo :**
- **Secure transfer required** : ✅ Włączyć (zalecane)
- **Allow Blob public access** : ❌ Wyłączyć (bezpieczeństwo)

**Data Lake Storage Gen2 :**
- **Hierarchical namespace** : ✅ Włączyć jeśli potrzeba Data Lake

### Krok 4 : Utworzyć konto

1. Kliknąć "Review + create"
2. Sprawdzić konfigurację
3. Kliknąć "Create"
4. Czekać na utworzenie (1-2 minuty)

**⚠️ Ważne :** Zanotować nazwę konta przechowywania.

---

## Blob Storage

### Czym jest Blob Storage?

**Blob Storage** = Przechowywanie obiektów dla plików

- **Containers** : Organizują pliki (jak foldery)
- **Blobs** : Pojedyncze pliki
- **Typy** : Block blobs, Page blobs, Append blobs

### Utworzyć container

**Przez portal Azure :**

1. Storage account → "Containers"
2. Kliknąć "+ Container"
3. Nazwa : `raw-data` (lub inna)
4. Poziom dostępu publicznego : Private (zalecane)
5. Kliknąć "Create"

**Przez Azure CLI :**

```bash
az storage container create \
  --name raw-data \
  --account-name mydataanalyststorage \
  --auth-mode login
```

**Przez Python :**

```python
from azure.storage.blob import BlobServiceClient

# Połączenie
connection_string = "DefaultEndpointsProtocol=https;AccountName=..."
blob_service_client = BlobServiceClient.from_connection_string(connection_string)

# Utworzyć container
container_client = blob_service_client.create_container("raw-data")
```

### Typy blobów

**Block Blobs :**
- Pliki (CSV, JSON, Parquet, obrazy, itp.)
- Do 4.75 TB na blob
- Zalecane dla większości przypadków

**Page Blobs :**
- Dyski wirtualne
- Do 8 TB

**Append Blobs :**
- Logi
- Dane tylko do dodawania

---

## Data Lake Storage Gen2

### Czym jest Data Lake Storage Gen2?

**Data Lake Storage Gen2** = Blob Storage + hierarchiczny system plików

- **Zgodne z Blob Storage** : Używa tych samych API
- **System plików** : Organizacja hierarchiczna
- **Zoptymalizowane Big Data** : Dla analytics i ML
- **Integracja** : Z Azure Synapse, Databricks, itp.

### Włączyć Data Lake Storage Gen2

**Podczas tworzenia konta :**
1. W "Advanced" → Włączyć "Hierarchical namespace"
2. Utworzyć konto

**⚠️ Ważne :** Nie można włączyć po utworzeniu.

### Struktura Data Lake

```
data-lake/
├── raw/
│   ├── 2024/
│   │   ├── 01/
│   │   └── 02/
├── processed/
│   └── 2024/
└── analytics/
    └── results/
```

### Tworzyć pliki i foldery

**Przez portal Azure :**

1. Storage account → "Data Lake"
2. Nawigować w strukturze
3. Przesyłać pliki
4. Tworzyć foldery

**Przez Python :**

```python
from azure.storage.filedatalake import DataLakeServiceClient

# Połączenie
account_name = "mydataanalyststorage"
account_key = "..."
datalake_service_client = DataLakeServiceClient(
    account_url=f"https://{account_name}.dfs.core.windows.net",
    credential=account_key
)

# Utworzyć system plików
file_system_client = datalake_service_client.create_file_system("data-lake")

# Utworzyć katalog
directory_client = file_system_client.create_directory("raw/2024")
```

---

## Przesyłać i zarządzać plikami

### Przesłać plik

**Przez portal Azure :**

1. Container → "Upload"
2. Wybrać plik
3. Kliknąć "Upload"

**Przez Azure CLI :**

```bash
az storage blob upload \
  --account-name mydataanalyststorage \
  --container-name raw-data \
  --name data.csv \
  --file ./local-data.csv \
  --auth-mode login
```

**Przez Python :**

```python
from azure.storage.blob import BlobServiceClient

blob_service_client = BlobServiceClient.from_connection_string(connection_string)
container_client = blob_service_client.get_container_client("raw-data")

# Przesłać plik
with open("local-data.csv", "rb") as data:
    container_client.upload_blob(name="data.csv", data=data)
```

### Pobrać plik

**Przez Python :**

```python
# Pobrać blob
blob_client = container_client.get_blob_client("data.csv")
with open("downloaded-data.csv", "wb") as download_file:
    download_file.write(blob_client.download_blob().readall())
```

### Listować pliki

**Przez Python :**

```python
# Listować wszystkie bloby w kontenerze
blob_list = container_client.list_blobs()
for blob in blob_list:
    print(f"Name: {blob.name}, Size: {blob.size}")
```

### Usunąć plik

**Przez Python :**

```python
# Usunąć blob
blob_client = container_client.get_blob_client("data.csv")
blob_client.delete_blob()
```

---

## Integracja z innymi usługami

### Azure Storage + Data Factory

**Użycie :**
- Źródło danych dla pipeline'ów ETL
- Miejsce docelowe dla przekształconych danych

**Przykład :**
```json
{
  "type": "AzureBlobStorage",
  "typeProperties": {
    "connectionString": "...",
    "container": "raw-data"
  }
}
```

### Azure Storage + Azure SQL Database

**Użycie :**
- Importować dane z Blob Storage
- Eksportować dane do Blob Storage

**Przykład SQL :**
```sql
-- Importować z Blob Storage
BULK INSERT my_table
FROM 'https://mystorageaccount.blob.core.windows.net/raw-data/data.csv'
WITH (
    FORMAT = 'CSV',
    FIRSTROW = 2
);
```

### Azure Storage + PowerBI

**Użycie :**
- Połączyć PowerBI z Blob Storage
- Analizować pliki bezpośrednio

**Konfiguracja :**
1. PowerBI → "Get Data"
2. "Azure Blob Storage"
3. Wprowadzić URL kontenera
4. Wybrać pliki

### Azure Storage + Azure Functions

**Użycie :**
- Wyzwalać Functions przy przesłaniu
- Automatycznie przetwarzać pliki

**Konfiguracja :**
1. Function → "Add trigger"
2. "Azure Blob Storage trigger"
3. Skonfigurować kontener i ścieżkę

---

## Dobre praktyki

### Organizacja

1. **Używać kontenerów** aby organizować według projektu
2. **Nazywać jasno** pliki i kontenery
3. **Organizować według daty** : `raw/2024/01/data.csv`
4. **Rozdzielać według typu** : `raw/`, `processed/`, `analytics/`

### Wydajność

1. **Używać losowych nazw** dla blobów (unikać sekwencji)
2. **Włączyć CDN** jeśli potrzeba globalnej dystrybucji (płatne)
3. **Używać blobów blokowych** dla większości przypadków
4. **Partycjonować dane** aby poprawić wydajność

### Koszty

1. **Monitorować użycie** w Azure Cost Management
2. **Usuwać niepotrzebne pliki**
3. **Używać odpowiednich klas** przechowywania
4. **Konfigurować reguły cyklu życia** aby automatyzować

### Bezpieczeństwo

1. **Nigdy nie udostępniać publicznie** kontenerów (oprócz konkretnej potrzeby)
2. **Używać SAS (Shared Access Signature)** dla tymczasowego dostępu
3. **Włączyć szyfrowanie** domyślnie
4. **Używać Azure AD** do uwierzytelniania

---

## Przykłady praktyczne

### Przykład 1 : Przesłać plik CSV

```python
from azure.storage.blob import BlobServiceClient
import pandas as pd

# Połączenie
connection_string = "DefaultEndpointsProtocol=https;AccountName=..."
blob_service_client = BlobServiceClient.from_connection_string(connection_string)
container_client = blob_service_client.get_container_client("raw-data")

# Czytać lokalny plik
df = pd.read_csv("local-data.csv")

# Przesłać do Azure Storage
with open("local-data.csv", "rb") as data:
    container_client.upload_blob(name="2024/01/data.csv", data=data)
```

### Przykład 2 : Pobrać i przetworzyć

```python
# Pobrać z Azure Storage
blob_client = container_client.get_blob_client("2024/01/data.csv")
with open("downloaded-data.csv", "wb") as download_file:
    download_file.write(blob_client.download_blob().readall())

# Przetworzyć
df = pd.read_csv("downloaded-data.csv")
# ... przetwarzanie ...

# Przesłać wynik
df.to_csv("processed-data.csv", index=False)
with open("processed-data.csv", "rb") as data:
    container_client.upload_blob(name="processed/2024/01/data.csv", data=data)
```

### Przykład 3 : Listować i filtrować

```python
# Listować wszystkie pliki w prefiksie
blob_list = container_client.list_blobs(name_starts_with="2024/01/")
for blob in blob_list:
    print(f"File: {blob.name}, Size: {blob.size} bytes, Modified: {blob.last_modified}")
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **Azure Storage = Nieograniczone przechowywanie** i wysoka dostępność
2. **Free Tier : 5 GB** przez 12 miesięcy
3. **Blob Storage** dla plików, **Data Lake Gen2** dla Big Data
4. **Organizować z kontenerami** i prefiksami
5. **Natywna integracja** ze wszystkimi usługami Azure data

## 🔗 Następny moduł

Przejdź do modułu [3. Azure Data Factory - ETL w chmurze](../03-data-factory/README.md), aby nauczyć się tworzyć pipeline'y ETL na Azure.

