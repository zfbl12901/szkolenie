# 7. Projets pratiques Azure

## 🎯 Objectifs

- Appliquer les connaissances acquises
- Créer des pipelines ETL complets
- Intégrer avec PowerBI
- Créer des projets pour votre portfolio
- Utiliser plusieurs services Azure

## 📋 Table des matières

1. [Projet 1 : Pipeline ETL Blob → SQL Database](#projet-1--pipeline-etl-blob---sql-database)
2. [Projet 2 : Data Lake avec Synapse](#projet-2--data-lake-avec-synapse)
3. [Projet 3 : Analytics avec PowerBI](#projet-3--analytics-avec-powerbi)
4. [Projet 4 : Pipeline automatisé complet](#projet-4--pipeline-automatisé-complet)
5. [Bonnes pratiques pour portfolio](#bonnes-pratiques-pour-portfolio)

---

## Projet 1 : Pipeline ETL Blob → SQL Database

### Objectif

Créer un pipeline ETL qui charge des fichiers CSV depuis Blob Storage vers SQL Database.

### Architecture

```
Blob Storage (CSV) → Data Factory → SQL Database → PowerBI
```

### Étapes

#### 1. Préparer les données

**Créer un container Blob Storage :**
- Nom : `raw-data`
- Uploader un fichier CSV de test

**Exemple de données CSV :**
```csv
id,name,email,created_at,status
1,John Doe,john@example.com,2024-01-01,active
2,Jane Smith,jane@example.com,2024-01-02,inactive
```

#### 2. Créer une base SQL Database

1. Portail Azure → Créer SQL Database
2. Configuration :
   - Name : `analytics-db`
   - Server : Créer nouveau serveur
   - Service tier : Basic (gratuit 12 mois)
3. Créer la base

#### 3. Créer la table dans SQL Database

```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    created_at DATETIME2,
    status VARCHAR(20)
);
```

#### 4. Créer un pipeline Data Factory

1. Data Factory Studio → "Author" → "Pipelines"
2. Créer un nouveau pipeline : `LoadCSVToSQL`
3. Ajouter activité "Copy Data"
4. Configuration :
   - **Source** : Azure Blob Storage (CSV)
   - **Sink** : Azure SQL Database (table users)
5. Publier le pipeline

#### 5. Exécuter le pipeline

1. Cliquer sur "Trigger now"
2. Vérifier l'exécution dans "Monitor"
3. Vérifier les données dans SQL Database

### Résultat

- Fichiers CSV chargés dans SQL Database
- Pipeline ETL fonctionnel
- Prêt pour analytics avec PowerBI

---

## Projet 2 : Data Lake avec Synapse

### Objectif

Créer un Data Lake complet avec ingestion, transformation et analytics.

### Architecture

```
Sources → Data Lake Storage (Raw) → Synapse (Transform) → Data Lake (Processed) → PowerBI
                ↓
        Data Factory (Orchestration)
```

### Étapes

#### 1. Structure Data Lake Storage

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

#### 2. Créer un workspace Synapse

1. Portail Azure → Créer Azure Synapse Analytics
2. Configuration :
   - Workspace name : `my-synapse-workspace`
   - Data Lake Storage : Créer nouveau
3. Créer le workspace

#### 3. Pipelines Data Factory pour transformation

**Pipeline pour users :**

1. Synapse Studio → "Integrate" → "Pipelines"
2. Créer pipeline : `TransformUsers`
3. Activités :
   - Source : Data Lake Storage (raw/users/)
   - Data Flow : Transformer (filtrer, nettoyer)
   - Sink : Data Lake Storage (processed/users/)
4. Publier

#### 4. Tables Synapse pour analytics

```sql
-- Créer une table externe
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

-- Requête analytique
SELECT 
    YEAR(created_at) AS year,
    COUNT(*) AS user_count
FROM users_processed
GROUP BY YEAR(created_at);
```

#### 5. Automatisation avec Triggers

1. Pipeline → "Add trigger" → "New/Edit"
2. Type : Schedule
3. Recurrence : Daily
4. Start time : 02:00
5. Sauvegarder

### Résultat

- Data Lake fonctionnel
- Pipeline automatisé
- Analytics avec Synapse
- Projet complet pour portfolio

---

## Projet 3 : Analytics avec PowerBI

### Objectif

Créer un système d'analytics complet avec PowerBI connecté à Azure.

### Étapes

#### 1. Préparer les données

**Dans SQL Database ou Synapse :**
- Charger des données
- Créer des vues analytiques

#### 2. Connecter PowerBI à Azure SQL Database

1. PowerBI Desktop → "Get Data"
2. "Azure" → "Azure SQL Database"
3. Configuration :
   - Server : `my-sql-server.database.windows.net`
   - Database : `analytics-db`
   - Authentication : Database
4. Sélectionner les tables ou vues
5. Cliquer sur "Load"

#### 3. Créer des visualisations

**Exemple :**
1. Importer la table `users`
2. Créer un graphique : Nombre d'utilisateurs par mois
3. Ajouter des filtres
4. Créer un dashboard

#### 4. Publier sur PowerBI Service

1. PowerBI Desktop → "Publish"
2. Sélectionner l'espace de travail
3. Publier
4. Accéder au rapport sur powerbi.com

#### 5. Actualiser les données

1. PowerBI Service → Dataset → "Schedule refresh"
2. Configuration :
   - Frequency : Daily
   - Time : 03:00
3. Sauvegarder

### Résultat

- Analytics avec PowerBI
- Visualisations interactives
- Actualisation automatique
- Projet complet pour portfolio

---

## Projet 4 : Pipeline automatisé complet

### Objectif

Créer un pipeline ETL complètement automatisé avec plusieurs services Azure.

### Architecture complète

```
Fichier CSV uploadé → Blob Storage (raw/)
    ↓ (Event)
Azure Function (Validation)
    ↓
Blob Storage (validated/)
    ↓ (Trigger)
Data Factory Pipeline (Transform CSV → Parquet)
    ↓
Data Lake Storage (processed/)
    ↓
Synapse (Analytics)
    ↓
SQL Database (Results)
    ↓
PowerBI (Visualization)
```

### Implémentation

#### 1. Azure Function de validation

```python
import azure.functions as func
import logging
import csv
from azure.storage.blob import BlobServiceClient

def main(blob: func.InputStream):
    logging.info(f'Processing blob: {blob.name}')
    
    # Lire le blob
    content = blob.read().decode('utf-8')
    reader = csv.DictReader(content.splitlines())
    
    # Valider
    valid_rows = []
    for row in reader:
        if row.get('email') and '@' in row['email']:
            valid_rows.append(row)
    
    # Uploader les données validées
    if valid_rows:
        # Upload vers validated/
        # ...
    
    logging.info(f'Validated {len(valid_rows)} rows')
```

#### 2. Data Factory Pipeline de transformation

**Pipeline :**
1. Source : Blob Storage (validated/)
2. Data Flow : Transformer (nettoyer, enrichir)
3. Sink : Data Lake Storage (processed/parquet/)

#### 3. Synapse pour analytics

```sql
-- Créer une vue analytique
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

#### 4. PowerBI pour visualisation

1. Connecter PowerBI à Synapse
2. Utiliser la vue `vw_user_analytics`
3. Créer des visualisations
4. Publier le rapport

### Résultat

- Pipeline complètement automatisé
- Validation automatique
- Transformation automatique
- Analytics disponibles immédiatement
- Visualisations PowerBI

---

## Bonnes pratiques pour portfolio

### Documentation

**Créer un README pour chaque projet :**

```markdown
# Projet : Pipeline ETL Azure

## Description
Pipeline ETL automatisé pour transformer des données CSV en Parquet.

## Architecture
- Blob Storage : Stockage
- Data Factory : Transformation
- SQL Database : Base de données
- PowerBI : Visualisation

## Résultats
- Réduction des coûts de 50%
- Temps de traitement réduit de 70%
```

### Visualisations

**Créer des diagrammes :**
- Architecture du système
- Flux de données
- Schéma de données

**Outils :**
- Draw.io
- Lucidchart
- Diagrammes ASCII dans README

### Métriques

**Inclure des métriques :**
- Temps d'exécution avant/après
- Coûts avant/après
- Volume de données traitées
- Performance des requêtes

### Code

**Bonnes pratiques :**
- Code commenté
- Variables d'environnement pour configuration
- Gestion d'erreurs
- Logging

### GitHub

**Créer un repository :**
- README avec documentation
- Scripts Data Factory (JSON)
- Scripts SQL
- Configuration
- Diagrammes

---

## 📊 Points clés à retenir

1. **Projets pratiques** : Essentiels pour portfolio
2. **Documentation** : Expliquer l'architecture et les résultats
3. **Métriques** : Montrer l'impact (performance, coûts)
4. **Code propre** : Commenté et organisé
5. **GitHub** : Partager vos projets

## 🔗 Ressources

- [Azure Architecture Center](https://learn.microsoft.com/azure/architecture/)
- [Azure Solutions](https://azure.microsoft.com/solutions/)
- [GitHub Azure Examples](https://github.com/Azure-Samples)

---

**Félicitations !** Vous avez terminé la formation Azure pour Data Analyst. Vous pouvez maintenant créer des projets complets sur Azure en utilisant les ressources gratuites disponibles.

