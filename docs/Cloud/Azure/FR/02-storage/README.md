# 2. Azure Storage - Stockage de données

## 🎯 Objectifs

- Comprendre Azure Storage et son utilisation
- Créer des comptes de stockage
- Utiliser Blob Storage et Data Lake Storage
- Uploader et gérer des fichiers
- Organiser les données

## 📋 Table des matières

1. [Introduction à Azure Storage](#introduction-à-azure-storage)
2. [Créer un compte de stockage](#créer-un-compte-de-stockage)
3. [Blob Storage](#blob-storage)
4. [Data Lake Storage Gen2](#data-lake-storage-gen2)
5. [Uploader et gérer des fichiers](#uploader-et-gérer-des-fichiers)
6. [Intégration avec autres services](#intégration-avec-autres-services)

---

## Introduction à Azure Storage

### Qu'est-ce qu'Azure Storage ?

**Azure Storage** = Service de stockage cloud géré

- **Stockage illimité** : Évolutif selon les besoins
- **Haute disponibilité** : 99.99% de disponibilité
- **Sécurisé** : Chiffrement par défaut
- **Intégration** : Avec tous les services Azure

### Types de stockage

1. **Blob Storage** : Fichiers (CSV, JSON, Parquet, etc.)
2. **Data Lake Storage Gen2** : Data Lake avec système de fichiers hiérarchique
3. **File Storage** : Partages de fichiers
4. **Queue Storage** : Files d'attente
5. **Table Storage** : Stockage NoSQL

### Free Tier Azure Storage

**Gratuit 12 mois :**
- 5 Go de stockage Blob
- 5 Go de stockage File
- 5 Go de stockage Table
- 5 Go de stockage Queue

**Gratuit à vie :**
- 200 Go de transfert de données sortantes/mois

**⚠️ Important :** Au-delà de ces limites, facturation normale.

---

## Créer un compte de stockage

### Étape 1 : Accéder à Azure Storage

1. Portail Azure → Rechercher "Storage accounts"
2. Cliquer sur "Storage accounts"
3. Cliquer sur "Create"

### Étape 2 : Configuration de base

**Informations de base :**
- **Subscription** : Choisir votre abonnement
- **Resource group** : Créer ou utiliser existant
- **Storage account name** : Nom unique globalement (ex: `mydataanalyststorage`)
- **Region** : Choisir la région la plus proche (ex: `France Central`)

**Options de performance :**
- **Performance** : Standard (recommandé pour débuter)
- **Redundancy** : LRS (Local Redundant Storage) - le moins cher

### Étape 3 : Options avancées

**Sécurité :**
- **Secure transfer required** : ✅ Activer (recommandé)
- **Allow Blob public access** : ❌ Désactiver (sécurité)

**Data Lake Storage Gen2 :**
- **Hierarchical namespace** : ✅ Activer si besoin de Data Lake

### Étape 4 : Créer le compte

1. Cliquer sur "Review + create"
2. Vérifier la configuration
3. Cliquer sur "Create"
4. Attendre la création (1-2 minutes)

**⚠️ Important :** Noter le nom du compte de stockage.

---

## Blob Storage

### Qu'est-ce que Blob Storage ?

**Blob Storage** = Stockage d'objets pour fichiers

- **Containers** : Organisent les fichiers (comme des dossiers)
- **Blobs** : Fichiers individuels
- **Types** : Block blobs, Page blobs, Append blobs

### Créer un container

**Via le portail Azure :**

1. Storage account → "Containers"
2. Cliquer sur "+ Container"
3. Nom : `raw-data` (ou autre)
4. Public access level : Private (recommandé)
5. Cliquer sur "Create"

**Via Azure CLI :**

```bash
az storage container create \
  --name raw-data \
  --account-name mydataanalyststorage \
  --auth-mode login
```

**Via Python :**

```python
from azure.storage.blob import BlobServiceClient

# Connexion
connection_string = "DefaultEndpointsProtocol=https;AccountName=..."
blob_service_client = BlobServiceClient.from_connection_string(connection_string)

# Créer un container
container_client = blob_service_client.create_container("raw-data")
```

### Types de blobs

**Block Blobs :**
- Fichiers (CSV, JSON, Parquet, images, etc.)
- Jusqu'à 4.75 To par blob
- Recommandé pour la plupart des cas

**Page Blobs :**
- Disques virtuels
- Jusqu'à 8 To

**Append Blobs :**
- Logs
- Données d'ajout uniquement

---

## Data Lake Storage Gen2

### Qu'est-ce que Data Lake Storage Gen2 ?

**Data Lake Storage Gen2** = Blob Storage + système de fichiers hiérarchique

- **Compatible Blob Storage** : Utilise les mêmes APIs
- **Système de fichiers** : Organisation hiérarchique
- **Optimisé Big Data** : Pour analytics et ML
- **Intégration** : Avec Azure Synapse, Databricks, etc.

### Activer Data Lake Storage Gen2

**Lors de la création du compte :**
1. Dans "Advanced" → Activer "Hierarchical namespace"
2. Créer le compte

**⚠️ Important :** Ne peut pas être activé après création.

### Structure Data Lake

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

### Créer des fichiers et dossiers

**Via le portail Azure :**

1. Storage account → "Data Lake"
2. Naviguer dans la structure
3. Uploader des fichiers
4. Créer des dossiers

**Via Python :**

```python
from azure.storage.filedatalake import DataLakeServiceClient

# Connexion
account_name = "mydataanalyststorage"
account_key = "..."
datalake_service_client = DataLakeServiceClient(
    account_url=f"https://{account_name}.dfs.core.windows.net",
    credential=account_key
)

# Créer un système de fichiers
file_system_client = datalake_service_client.create_file_system("data-lake")

# Créer un répertoire
directory_client = file_system_client.create_directory("raw/2024")
```

---

## Uploader et gérer des fichiers

### Uploader un fichier

**Via le portail Azure :**

1. Container → "Upload"
2. Sélectionner le fichier
3. Cliquer sur "Upload"

**Via Azure CLI :**

```bash
az storage blob upload \
  --account-name mydataanalyststorage \
  --container-name raw-data \
  --name data.csv \
  --file ./local-data.csv \
  --auth-mode login
```

**Via Python :**

```python
from azure.storage.blob import BlobServiceClient

blob_service_client = BlobServiceClient.from_connection_string(connection_string)
container_client = blob_service_client.get_container_client("raw-data")

# Uploader un fichier
with open("local-data.csv", "rb") as data:
    container_client.upload_blob(name="data.csv", data=data)
```

### Télécharger un fichier

**Via Python :**

```python
# Télécharger un blob
blob_client = container_client.get_blob_client("data.csv")
with open("downloaded-data.csv", "wb") as download_file:
    download_file.write(blob_client.download_blob().readall())
```

### Lister les fichiers

**Via Python :**

```python
# Lister tous les blobs dans un container
blob_list = container_client.list_blobs()
for blob in blob_list:
    print(f"Name: {blob.name}, Size: {blob.size}")
```

### Supprimer un fichier

**Via Python :**

```python
# Supprimer un blob
blob_client = container_client.get_blob_client("data.csv")
blob_client.delete_blob()
```

---

## Intégration avec autres services

### Azure Storage + Data Factory

**Utilisation :**
- Source de données pour pipelines ETL
- Destination pour données transformées

**Exemple :**
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

**Utilisation :**
- Importer des données depuis Blob Storage
- Exporter des données vers Blob Storage

**Exemple SQL :**
```sql
-- Importer depuis Blob Storage
BULK INSERT my_table
FROM 'https://mystorageaccount.blob.core.windows.net/raw-data/data.csv'
WITH (
    FORMAT = 'CSV',
    FIRSTROW = 2
);
```

### Azure Storage + PowerBI

**Utilisation :**
- Connecter PowerBI à Blob Storage
- Analyser des fichiers directement

**Configuration :**
1. PowerBI → "Get Data"
2. "Azure Blob Storage"
3. Entrer l'URL du container
4. Sélectionner les fichiers

### Azure Storage + Azure Functions

**Utilisation :**
- Déclencher Functions lors d'upload
- Traiter automatiquement les fichiers

**Configuration :**
1. Function → "Add trigger"
2. "Azure Blob Storage trigger"
3. Configurer le container et le chemin

---

## Bonnes pratiques

### Organisation

1. **Utiliser des containers** pour organiser par projet
2. **Nommer clairement** les fichiers et containers
3. **Organiser par date** : `raw/2024/01/data.csv`
4. **Séparer par type** : `raw/`, `processed/`, `analytics/`

### Performance

1. **Utiliser des noms aléatoires** pour les blobs (éviter les séquences)
2. **Activer CDN** si besoin de distribution globale (payant)
3. **Utiliser des blobs de blocs** pour la plupart des cas
4. **Partitionner les données** pour améliorer les performances

### Coûts

1. **Surveiller l'utilisation** dans Azure Cost Management
2. **Supprimer les fichiers inutiles**
3. **Utiliser les bonnes classes** de stockage
4. **Configurer des règles de cycle de vie** pour automatiser

### Sécurité

1. **Ne jamais rendre publics** les containers (sauf besoin spécifique)
2. **Utiliser SAS (Shared Access Signature)** pour accès temporaire
3. **Activer le chiffrement** par défaut
4. **Utiliser Azure AD** pour authentification

---

## Exemples pratiques

### Exemple 1 : Uploader un fichier CSV

```python
from azure.storage.blob import BlobServiceClient
import pandas as pd

# Connexion
connection_string = "DefaultEndpointsProtocol=https;AccountName=..."
blob_service_client = BlobServiceClient.from_connection_string(connection_string)
container_client = blob_service_client.get_container_client("raw-data")

# Lire un fichier local
df = pd.read_csv("local-data.csv")

# Uploader vers Azure Storage
with open("local-data.csv", "rb") as data:
    container_client.upload_blob(name="2024/01/data.csv", data=data)
```

### Exemple 2 : Télécharger et traiter

```python
# Télécharger depuis Azure Storage
blob_client = container_client.get_blob_client("2024/01/data.csv")
with open("downloaded-data.csv", "wb") as download_file:
    download_file.write(blob_client.download_blob().readall())

# Traiter
df = pd.read_csv("downloaded-data.csv")
# ... traitement ...

# Uploader le résultat
df.to_csv("processed-data.csv", index=False)
with open("processed-data.csv", "rb") as data:
    container_client.upload_blob(name="processed/2024/01/data.csv", data=data)
```

### Exemple 3 : Lister et filtrer

```python
# Lister tous les fichiers dans un préfixe
blob_list = container_client.list_blobs(name_starts_with="2024/01/")
for blob in blob_list:
    print(f"File: {blob.name}, Size: {blob.size} bytes, Modified: {blob.last_modified}")
```

---

## 📊 Points clés à retenir

1. **Azure Storage = Stockage illimité** et hautement disponible
2. **Free Tier : 5 Go** pendant 12 mois
3. **Blob Storage** pour fichiers, **Data Lake Gen2** pour Big Data
4. **Organiser avec containers** et préfixes
5. **Intégration native** avec tous les services Azure data

## 🔗 Prochain module

Passer au module [3. Azure Data Factory - ETL Cloud](../03-data-factory/README.md) pour apprendre à créer des pipelines ETL sur Azure.

