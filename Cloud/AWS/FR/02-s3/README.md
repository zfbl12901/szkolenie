# 2. Amazon S3 - Stockage de données

## 🎯 Objectifs

- Comprendre Amazon S3 et son utilisation
- Créer et gérer des buckets S3
- Uploader et organiser des fichiers
- Comprendre les classes de stockage
- Intégrer S3 avec d'autres services AWS

## 📋 Table des matières

1. [Introduction à S3](#introduction-à-s3)
2. [Créer un bucket S3](#créer-un-bucket-s3)
3. [Uploader et gérer des fichiers](#uploader-et-gérer-des-fichiers)
4. [Classes de stockage](#classes-de-stockage)
5. [Organisation des données](#organisation-des-données)
6. [Intégration avec autres services](#intégration-avec-autres-services)

---

## Introduction à S3

### Qu'est-ce qu'Amazon S3 ?

**Amazon S3** (Simple Storage Service) = Service de stockage d'objets

- Stockage illimité
- Haute disponibilité (99.99%)
- Sécurisé par défaut
- Intégration avec tous les services AWS

### Cas d'usage pour Data Analyst

- **Data Lake** : Stocker des données brutes
- **Backup** : Sauvegarder des données
- **ETL** : Source/destination pour pipelines
- **Analytics** : Données pour Athena, Redshift
- **Archivage** : Données historiques

### Free Tier S3

**Gratuit à vie :**
- 5 Go de stockage standard
- 20 000 requêtes GET
- 2 000 requêtes PUT
- 15 Go de transfert de données sortantes

**⚠️ Important :** Au-delà de ces limites, facturation normale.

---

## Créer un bucket S3

### Étape 1 : Accéder à S3

1. Console AWS → Rechercher "S3"
2. Cliquer sur "Amazon S3"
3. Cliquer sur "Create bucket"

### Étape 2 : Configuration du bucket

**Informations de base :**
- **Bucket name** : Nom unique globalement (ex: `my-data-analyst-bucket`)
- **Region** : Choisir la région la plus proche (ex: `eu-west-3` Paris)

**Options de configuration :**

1. **Object Ownership**
   - "ACLs disabled" (recommandé)
   - "Bucket owner enforced"

2. **Block Public Access**
   - ✅ **Tout activer** (sécurité par défaut)
   - Désactiver seulement si besoin spécifique

3. **Versioning**
   - Désactivé par défaut (gratuit)
   - Activer si besoin de versions multiples

4. **Tags** (optionnel)
   - Ajouter des tags pour organisation
   - Ex: `Project: Data-Analyst-Training`

5. **Default encryption**
   - ✅ Activer (recommandé)
   - "Amazon S3 managed keys (SSE-S3)" (gratuit)

### Étape 3 : Créer le bucket

1. Cliquer sur "Create bucket"
2. Bucket créé et visible dans la liste
3. Prêt à utiliser

**⚠️ Important :** Le nom du bucket doit être unique globalement dans AWS.

---

## Uploader et gérer des fichiers

### Uploader un fichier

**Méthode 1 : Interface web**

1. Cliquer sur le nom du bucket
2. Cliquer sur "Upload"
3. "Add files" ou "Add folder"
4. Sélectionner les fichiers
5. Cliquer sur "Upload"

**Méthode 2 : AWS CLI**

```bash
# Installer AWS CLI (si pas déjà fait)
# Windows: https://aws.amazon.com/cli/
# Linux/Mac: pip install awscli

# Configurer les credentials
aws configure

# Uploader un fichier
aws s3 cp local-file.csv s3://my-data-analyst-bucket/data/
```

**Méthode 3 : SDK Python (boto3)**

```python
import boto3

# Créer un client S3
s3 = boto3.client('s3')

# Uploader un fichier
s3.upload_file('local-file.csv', 'my-data-analyst-bucket', 'data/file.csv')
```

### Télécharger un fichier

**Interface web :**
1. Cliquer sur le fichier
2. Cliquer sur "Download"

**AWS CLI :**
```bash
aws s3 cp s3://my-data-analyst-bucket/data/file.csv local-file.csv
```

**Python :**
```python
s3.download_file('my-data-analyst-bucket', 'data/file.csv', 'local-file.csv')
```

### Gérer les fichiers

**Actions disponibles :**
- **Download** : Télécharger
- **Open** : Ouvrir dans le navigateur
- **Copy** : Copier vers un autre emplacement
- **Move** : Déplacer
- **Delete** : Supprimer
- **Make public** : Rendre public (attention sécurité)

---

## Classes de stockage

### S3 Standard (par défaut)

**Utilisation :**
- Données fréquemment accédées
- Applications en production

**Caractéristiques :**
- Accès rapide
- 99.99% de disponibilité
- Coût : ~0.023$ par Go/mois

**Free Tier :** 5 Go gratuit

### S3 Intelligent-Tiering

**Utilisation :**
- Données avec accès variable
- Optimisation automatique des coûts

**Caractéristiques :**
- Déplace automatiquement entre classes
- Pas de frais de récupération
- Coût : ~0.023$ par Go/mois

### S3 Standard-IA (Infrequent Access)

**Utilisation :**
- Données rarement accédées
- Backup, archives

**Caractéristiques :**
- Accès rapide quand nécessaire
- Coût stockage : ~0.0125$ par Go/mois
- Coût récupération : ~0.01$ par Go

### S3 One Zone-IA

**Utilisation :**
- Données reproductibles
- Backup secondaire

**Caractéristiques :**
- Stockage dans une seule zone
- Coût : ~0.01$ par Go/mois
- ⚠️ Risque de perte si zone défaillante

### S3 Glacier

**Utilisation :**
- Archivage long terme
- Données rarement nécessaires

**Caractéristiques :**
- Récupération : 1-5 minutes à plusieurs heures
- Coût : ~0.004$ par Go/mois
- Frais de récupération selon vitesse

### Choisir la classe de stockage

**Pour Data Analyst :**
- **S3 Standard** : Données actives (analyses fréquentes)
- **S3 Standard-IA** : Données historiques (analyses occasionnelles)
- **S3 Glacier** : Archives (rarement utilisées)

**Transition automatique :**
- Configurer des règles de transition
- Exemple : Standard → Standard-IA après 30 jours

---

## Organisation des données

### Structure recommandée

**Organisation par projet :**
```
bucket-name/
├── raw/              # Données brutes
│   ├── 2024/
│   │   ├── 01/
│   │   ├── 02/
│   │   └── ...
├── processed/        # Données transformées
│   ├── 2024/
│   └── ...
├── analytics/        # Données pour analyse
│   └── ...
└── archive/          # Archives
    └── ...
```

**Organisation par type :**
```
bucket-name/
├── csv/
├── json/
├── parquet/
└── logs/
```

### Préfixes et dossiers

**S3 n'a pas de "vrais" dossiers**, mais utilise des préfixes :

- `data/2024/01/file.csv` = Préfixe `data/2024/01/`
- Interface web simule des dossiers
- Utiliser `/` pour organiser

**Bonnes pratiques :**
- Utiliser des préfixes cohérents
- Inclure la date dans le chemin
- Séparer par type de données

---

## Intégration avec autres services

### S3 + AWS Glue

**Utilisation :**
- S3 comme source de données
- Glue transforme les données
- Résultat vers S3 ou autre destination

**Exemple :**
```python
# Job Glue lit depuis S3
datasource = glueContext.create_dynamic_frame.from_catalog(
    database = "my_database",
    table_name = "s3_data"
)
```

### S3 + Amazon Athena

**Utilisation :**
- Requêtes SQL directement sur fichiers S3
- Pas besoin de charger dans base de données
- Pay-per-query

**Exemple :**
```sql
-- Créer une table externe pointant vers S3
CREATE EXTERNAL TABLE my_table (
    id INT,
    name STRING
)
STORED AS PARQUET
LOCATION 's3://my-bucket/data/';
```

### S3 + Amazon Redshift

**Utilisation :**
- S3 comme source pour COPY
- Redshift comme data warehouse
- Chargement rapide de grandes quantités

**Exemple :**
```sql
COPY my_table
FROM 's3://my-bucket/data/file.csv'
IAM_ROLE 'arn:aws:iam::account:role/RedshiftRole'
CSV;
```

### S3 + AWS Lambda

**Utilisation :**
- Déclencher Lambda lors d'upload
- Traitement automatique des fichiers
- Transformation, validation, etc.

**Configuration :**
1. S3 → Properties → Event notifications
2. Créer une notification
3. Déclencher : "All object create events"
4. Destination : Lambda function

---

## Bonnes pratiques

### Sécurité

1. **Ne jamais rendre les buckets publics** (sauf besoin spécifique)
2. **Utiliser IAM** pour contrôler l'accès
3. **Activer le chiffrement** par défaut
4. **Utiliser des bucket policies** pour permissions granulaires

### Performance

1. **Utiliser des préfixes** pour distribuer la charge
2. **Éviter les noms séquentiels** (ex: file1, file2, file3)
3. **Utiliser Multipart Upload** pour gros fichiers (>100MB)
4. **Activer Transfer Acceleration** si besoin (payant)

### Coûts

1. **Surveiller l'utilisation** régulièrement
2. **Utiliser les bonnes classes** de stockage
3. **Supprimer les fichiers inutiles**
4. **Configurer des transitions** automatiques
5. **Utiliser S3 Lifecycle** pour automatiser

### Organisation

1. **Nommer les buckets** de manière cohérente
2. **Utiliser des tags** pour organisation
3. **Documenter la structure** des données
4. **Créer des conventions** de nommage

---

## Exemples pratiques

### Exemple 1 : Uploader un fichier CSV

```python
import boto3
import pandas as pd

# Créer un client S3
s3 = boto3.client('s3')

# Lire un fichier local
df = pd.read_csv('data.csv')

# Uploader vers S3
s3.upload_file('data.csv', 'my-bucket', 'raw/2024/data.csv')
```

### Exemple 2 : Lister les fichiers d'un préfixe

```python
# Lister tous les fichiers dans un préfixe
response = s3.list_objects_v2(
    Bucket='my-bucket',
    Prefix='raw/2024/'
)

for obj in response.get('Contents', []):
    print(obj['Key'], obj['Size'])
```

### Exemple 3 : Télécharger et traiter

```python
# Télécharger depuis S3
s3.download_file('my-bucket', 'raw/data.csv', 'local-data.csv')

# Traiter
df = pd.read_csv('local-data.csv')
# ... traitement ...

# Uploader le résultat
df.to_csv('processed-data.csv', index=False)
s3.upload_file('processed-data.csv', 'my-bucket', 'processed/data.csv')
```

---

## 📊 Points clés à retenir

1. **S3 = Stockage illimité** et hautement disponible
2. **Free Tier : 5 Go** toujours gratuit
3. **Organiser avec préfixes** pour meilleure performance
4. **Choisir la bonne classe** selon l'usage
5. **S3 s'intègre** avec tous les services AWS data

## 🔗 Prochain module

Passer au module [3. AWS Glue - ETL Serverless](../03-glue/README.md) pour apprendre à transformer des données avec AWS Glue.

