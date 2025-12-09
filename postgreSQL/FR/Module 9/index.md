# Module 9 — Atelier pratique & Certification interne

## 🎯 Objectifs
- Mettre en pratique l'ensemble des connaissances acquises dans la formation.
- Valider les compétences via un projet concret de type Data Engineer.

---

## 🛠 Atelier : construire un Mini Data Warehouse

### 1. Modélisation d'un Star Schema
- Identifier les **fact tables** et **dimension tables**.
- Définir les clés primaires et étrangères.
- Choisir entre normalisation/dénormalisation selon le workload.

---

### 2. Import d'un dataset massif
- Utiliser `COPY` ou `\copy` pour importer des millions de lignes.
- Vérifier l'intégrité des données.
- Préparer les tables pour le partitionnement.

---

### 3. Partitionnement & indexation intelligente
- Partitionner les tables volumineuses (`RANGE`, `LIST` ou `HASH`).
- Créer des indexes adaptés :
  - Multicolonnes pour les requêtes fréquentes
  - BRIN pour les colonnes séquentielles
  - Index partiels pour les filtres fréquents

---

### 4. Optimisation de 10 requêtes avec benchmarks
- Identifier les requêtes critiques.
- Mesurer les performances avec :
```sql
EXPLAIN (ANALYZE, BUFFERS) SELECT ...;
```
- Réécrire et optimiser chaque requête.
- Comparer les temps avant / après optimisation.

---

### 5. Analyse complète via explain.dalibo.com
- Générer le plan JSON :
```sql
EXPLAIN (ANALYZE, COSTS, VERBOSE, BUFFERS, FORMAT JSON) SELECT ...;
```
- Visualiser l'arbre pour détecter les nœuds coûteux.
- Corriger les problèmes détectés et comparer les améliorations.

---

### 6. Mise en place d'un petit pipeline ELT
- Ingestion des données brutes.
- Transformation en SQL pour préparer le Data Warehouse.
- Stockage final prêt pour l'analytics.

---

### 7. Monitoring & statistiques évolutives
- Suivre les performances avec `pg_stat_statements` et `pg_stat_activity`.
- Vérifier l'efficacité des index et la taille des tables.
- Ajuster autovacuum et autres paramètres si nécessaire.

---

### 8. Documentation & restitution orale
- Documenter :
  - Modèle de données
  - Choix d'indexation et partitionnement
  - Optimisations réalisées
- Restituer oralement la démarche, les résultats et les apprentissages.

---

## ✅ Compétences attendues à l'issue du module
- Construire un mini Data Warehouse complet.
- Importer et gérer un dataset massif.
- Appliquer partitionnement et indexation pour optimiser les requêtes.
- Analyser et optimiser les performances avec EXPLAIN et explain.dalibo.com.
- Mettre en place un pipeline ELT simple.
- Surveiller et maintenir le système en production.
- Documenter et présenter un projet analytique de manière professionnelle.
