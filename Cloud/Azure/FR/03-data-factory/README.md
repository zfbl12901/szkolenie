# 3. Azure Data Factory - ETL Cloud

## 🎯 Objectifs

- Comprendre Azure Data Factory et son rôle
- Créer des pipelines ETL
- Utiliser des activités de transformation
- Intégrer avec des sources de données
- Orchestrer des workflows

## 📋 Table des matières

1. [Introduction à Data Factory](#introduction-à-data-factory)
2. [Créer un Data Factory](#créer-un-data-factory)
3. [Créer un pipeline](#créer-un-pipeline)
4. [Activités de transformation](#activités-de-transformation)
5. [Intégration avec sources de données](#intégration-avec-sources-de-données)
6. [Orchestration et scheduling](#orchestration-et-scheduling)

---

## Introduction à Data Factory

### Qu'est-ce qu'Azure Data Factory ?

**Azure Data Factory** = Service ETL cloud géré

- **ETL** : Extract, Transform, Load
- **Cloud** : Pas d'infrastructure à gérer
- **Géré** : Microsoft gère l'infrastructure
- **Scalable** : S'adapte automatiquement

### Composants Data Factory

1. **Pipelines** : Flux de travail ETL
2. **Activities** : Étapes dans un pipeline
3. **Datasets** : Représentations des données
4. **Linked Services** : Connexions aux sources
5. **Triggers** : Déclenchement automatique

### Free Tier Data Factory

**Gratuit à vie :**
- 5 pipelines gratuits
- Activités limitées
- Au-delà : facturation à l'usage

**⚠️ Important :** Surveiller les coûts, surtout pour les activités de transformation.

---

## Créer un Data Factory

### Étape 1 : Accéder à Data Factory

1. Portail Azure → Rechercher "Data Factory"
2. Cliquer sur "Data factories"
3. Cliquer sur "Create"

### Étape 2 : Configuration de base

**Informations de base :**
- **Subscription** : Choisir votre abonnement
- **Resource group** : Créer ou utiliser existant
- **Name** : `my-data-factory` (unique globalement)
- **Version** : V2 (recommandé)
- **Region** : Choisir la région la plus proche

**Git configuration (optionnel) :**
- **Configure Git later** : Pour débuter rapidement
- Ou configurer Git/GitHub pour versioning

### Étape 3 : Créer le Data Factory

1. Cliquer sur "Review + create"
2. Vérifier la configuration
3. Cliquer sur "Create"
4. Attendre la création (2-3 minutes)

**⚠️ Important :** Noter le nom du Data Factory.

### Étape 4 : Ouvrir Data Factory Studio

1. Une fois créé, cliquer sur "Open Azure Data Factory Studio"
2. Interface web pour créer des pipelines

---

## Créer un pipeline

### Étape 1 : Créer un Linked Service

**Linked Service = Connexion à une source de données**

**Exemple : Azure Blob Storage**

1. Data Factory Studio → "Manage" → "Linked services"
2. Cliquer sur "+ New"
3. Rechercher "Azure Blob Storage"
4. Configuration :
   - **Name** : `AzureBlobStorage1`
   - **Storage account name** : Sélectionner votre compte
   - **Authentication method** : Account key (ou autre)
5. Cliquer sur "Create"

### Étape 2 : Créer un Dataset

**Dataset = Représentation des données**

1. Data Factory Studio → "Author" → "Datasets"
2. Cliquer sur "+ New"
3. Choisir "Azure Blob Storage"
4. Configuration :
   - **Name** : `CSVData`
   - **Linked service** : `AzureBlobStorage1`
   - **File path** : `raw-data/`
   - **File format** : DelimitedText (CSV)
5. Cliquer sur "Create"

### Étape 3 : Créer un pipeline

1. Data Factory Studio → "Author" → "Pipelines"
2. Cliquer sur "+ New pipeline"
3. Nommer le pipeline : `CopyCSVToParquet`

### Étape 4 : Ajouter une activité

**Exemple : Copy Data**

1. Dans le pipeline, glisser "Copy Data" depuis "Move & transform"
2. Configurer :
   - **Source** : Dataset `CSVData`
   - **Sink (Destination)** : Créer un nouveau dataset Parquet
3. Cliquer sur "Publish" pour sauvegarder

---

## Activités de transformation

### Copy Data

**Copier des données d'une source à une destination**

**Configuration :**
- **Source** : Dataset source
- **Sink** : Dataset destination
- **Mapping** : Mapping des colonnes

**Exemple : CSV → Parquet**

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

**Transformation de données avec interface graphique**

**Étapes :**
1. Créer un Data Flow
2. Ajouter une source
3. Ajouter des transformations :
   - **Select** : Sélectionner colonnes
   - **Filter** : Filtrer des lignes
   - **Derived Column** : Créer des colonnes calculées
   - **Aggregate** : Agrégations
   - **Join** : Joindre des données
4. Ajouter un sink

**Exemple de transformations :**

```
Source (CSV) 
  → Select (colonnes)
  → Filter (status = 'active')
  → Derived Column (nouvelle colonne)
  → Aggregate (SUM, COUNT)
  → Sink (Parquet)
```

### Lookup

**Rechercher des valeurs dans une autre source**

**Utilisation :**
- Valider des données
- Enrichir des données
- Vérifier des références

### Stored Procedure

**Exécuter une procédure stockée SQL**

**Utilisation :**
- Traitement dans SQL Database
- Logique métier complexe
- Optimisation côté base

---

## Intégration avec sources de données

### Azure Blob Storage

**Source de données :**

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

**Source de données :**

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

**Source de données :**

```json
{
  "type": "AzureBlobFS",
  "typeProperties": {
    "url": "https://account.dfs.core.windows.net",
    "fileSystem": "data-lake"
  }
}
```

### Fichiers locaux (via Self-hosted IR)

**Integration Runtime :**
- Self-hosted IR pour accès aux fichiers locaux
- Installer sur une machine locale
- Connecter au Data Factory

---

## Orchestration et scheduling

### Déclencher manuellement

1. Data Factory Studio → "Monitor"
2. Sélectionner le pipeline
3. Cliquer sur "Trigger now"
4. Voir l'exécution en temps réel

### Planifier un pipeline (Trigger)

**Créer un trigger :**

1. Pipeline → "Add trigger" → "New/Edit"
2. Type : "Schedule"
3. Configuration :
   - **Name** : `DailyTrigger`
   - **Type** : Schedule
   - **Recurrence** : Daily
   - **Start time** : 02:00
4. Cliquer sur "OK"

**Types de triggers :**
- **Schedule** : Planifié (cron)
- **Event** : Déclenché par événement
- **Tumbling window** : Fenêtre glissante

### Déclencher par événement

**Exemple : Nouveau fichier dans Blob Storage**

1. Créer un trigger "Storage event"
2. Configurer :
   - **Storage account** : Votre compte
   - **Container** : `raw-data`
   - **Event type** : Blob created
3. Associer au pipeline

---

## Bonnes pratiques

### Performance

1. **Utiliser Data Flow** pour transformations complexes
2. **Optimiser les activités** pour réduire le temps
3. **Utiliser le parallélisme** quand possible
4. **Choisir les bonnes régions** pour réduire la latence

### Coûts

1. **Surveiller les exécutions** dans Monitor
2. **Utiliser les 5 pipelines gratuits** intelligemment
3. **Optimiser les Data Flows** (coûteux)
4. **Arrêter les pipelines** non utilisés

### Organisation

1. **Nommer clairement** les pipelines et activités
2. **Documenter** les transformations
3. **Versionner** avec Git
4. **Tester** avant de publier

### Sécurité

1. **Utiliser Key Vault** pour les secrets
2. **Limiter les permissions** des Linked Services
3. **Auditer** les exécutions
4. **Chiffrer** les données en transit

---

## Exemples pratiques

### Exemple 1 : Pipeline simple CSV → Parquet

**Pipeline :**
1. Source : Azure Blob Storage (CSV)
2. Activity : Copy Data
3. Sink : Azure Blob Storage (Parquet)

**Configuration :**
- Source : `raw-data/data.csv`
- Sink : `processed-data/data.parquet`
- Format : DelimitedText → Parquet

### Exemple 2 : Pipeline avec transformation

**Pipeline :**
1. Source : Azure SQL Database
2. Data Flow :
   - Select colonnes
   - Filter lignes
   - Aggregate
3. Sink : Azure Blob Storage (Parquet)

### Exemple 3 : Pipeline orchestré

**Pipeline :**
1. Lookup : Vérifier si nouvelles données
2. If Condition : Si nouvelles données
3. Copy Data : Copier vers staging
4. Data Flow : Transformer
5. Copy Data : Charger dans destination

---

## 📊 Points clés à retenir

1. **Data Factory = ETL cloud** géré par Microsoft
2. **Free Tier : 5 pipelines** gratuits
3. **Pipelines** orchestrent les activités
4. **Data Flows** pour transformations complexes
5. **Triggers** permettent l'automatisation

## 🔗 Prochain module

Passer au module [4. Azure SQL Database - Base de données](../04-sql-database/README.md) pour apprendre à utiliser SQL Database sur Azure.

