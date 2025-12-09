# Module 6 — Modélisation & Data Engineering

## 🎯 Objectifs
- Structurer les données pour l'analytics, la scalabilité et les pipelines de données.

---

## 📚 Contenu

### 1. Modélisation BI : Star Schema / Snowflake
- **Star Schema** :
  - Table centrale (fact table) reliée aux dimensions.
  - Avantages : requêtes simples, bonnes performances sur agrégations.
- **Snowflake Schema** :
  - Dimensions normalisées.
  - Avantages : moins de redondance, meilleure intégrité.
- Comparaison et choix selon volumes et besoins analytiques.

---

### 2. Normalisation / dénormalisation
- **Normalisation** :
  - Éviter la duplication de données.
  - Garantir l'intégrité référentielle.
- **Dénormalisation** :
  - Optimisation des lectures pour l'analytics.
  - Stockage redondant parfois nécessaire pour les gros volumes.
- Stratégies hybrides selon workload OLAP vs OLTP.

---

### 3. Techniques pour big tables (100M – 1B rows)
- **Partitionnement** :
  - **Range**, **List**, **Hash**
  - Améliore performance des requêtes et maintenance.
- **Indexation adaptée** :
  - Multicolonnes
  - BRIN pour colonnes séquentielles
- Gestion des agrégations pré-calculées (materialized views).
- Stratégies de vacuum et autovacuum pour limiter le bloat.

---

### 4. Retention & archival strategies
- Archivage des données anciennes pour réduire la taille active :
  - Tables historiques
  - Partitions fermées
- Techniques de purge et suppression progressive.
- Importance de conserver un historique cohérent pour l'analytics.

---

### 5. Pipelines de données

#### 5.1 Ingestion
- Import direct avec `COPY` ou `\copy`.
- Ingestion batch vs streaming.
- Pré-traitement minimal dans PostgreSQL pour optimiser les performances.

#### 5.2 ELT / ETL en SQL
- Transformations réalisées **dans PostgreSQL** après ingestion.
- Avantages : réduction de flux externes, exploitation du moteur SQL.
- Bonnes pratiques :
  - Utiliser CTE, fonctions analytiques, index temporaires
  - Charger d'abord les données brutes, transformer ensuite.

#### 5.3 Change Data Capture (CDC)
- Capturer les modifications pour pipelines incrémentaux.
- Utilisation des slots logiques ou triggers pour alimenter ETL.
- Intégration avec outils externes (Kafka, Debezium).

#### 5.4 Logical replication
- Réplication sélective de tables pour :
  - Analytics temps réel
  - Migration
  - Haute disponibilité
- Concepts :
  - **Publication** : table à publier
  - **Subscription** : base réceptrice
```sql
CREATE PUBLICATION pub_users FOR TABLE users;
CREATE SUBSCRIPTION sub_users CONNECTION 'host=... dbname=... user=...' PUBLICATION pub_users;
```

---

## ✅ Compétences attendues à l'issue du module
- Concevoir des schémas BI adaptés à l'analytics et au reporting.
- Choisir entre normalisation et dénormalisation selon workload.
- Gérer efficacement des tables massives et leur indexation.
- Mettre en place des stratégies de retention et archivage.
- Construire des pipelines de données avec ingestion, transformation et CDC.
- Maîtriser la réplication logique pour la scalabilité et la haute disponibilité.
