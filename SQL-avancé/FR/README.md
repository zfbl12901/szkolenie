# Formation SQL/PostgreSQL Avancé - Optimisation des Requêtes

## 📚 Vue d'ensemble

Cette formation couvre les techniques avancées d'optimisation de requêtes SQL/PostgreSQL, avec un focus particulier sur l'utilisation de **Dalibo** pour l'analyse et l'optimisation des performances.

## 🎯 Objectifs pédagogiques

- Comprendre les mécanismes d'exécution des requêtes PostgreSQL
- Maîtriser les techniques d'optimisation avancées
- Utiliser Dalibo pour analyser et optimiser les performances
- Interpréter les indicateurs de performance clés
- Appliquer les bonnes pratiques dans des cas réels

## 📖 Structure de la formation

### 1. [Fondamentaux de l'optimisation](./01-fondamentaux/README.md)
   - Architecture PostgreSQL et planificateur de requêtes
   - Types d'index et leur utilisation
   - Statistiques et ANALYZE

### 2. [Analyse des plans d'exécution](./02-plans-execution/README.md)
   - EXPLAIN et EXPLAIN ANALYZE
   - Interprétation des opérations (Seq Scan, Index Scan, etc.)
   - Coûts et temps d'exécution

### 3. [Dalibo - Outil d'analyse](./03-dalibo/README.md)
   - Installation et configuration
   - Analyse de requêtes avec pg_stat_statements
   - Rapports de performance
   - Recommandations automatiques

### 4. [Indicateurs de performance](./04-indicateurs/README.md)
   - Métriques clés à surveiller
   - Interprétation des indicateurs Dalibo
   - Seuils d'alerte et bonnes pratiques

### 5. [Techniques d'optimisation](./05-techniques/README.md)
   - Optimisation des jointures
   - Optimisation des agrégations
   - Optimisation des sous-requêtes
   - Partitionnement et parallélisme

### 6. [Cas pratiques](./06-cas-pratiques/README.md)
   - Scénarios réels d'optimisation
   - Avant/Après avec métriques
   - Résolution de problèmes courants

### 7. [Exercices](./07-exercices/README.md)
   - Exercices guidés
   - Problèmes à résoudre
   - Solutions commentées

## 🚀 Démarrage rapide

1. **Prérequis**
   - PostgreSQL 12+ installé
   - Accès à une base de données de test
   - Extension `pg_stat_statements` activée

2. **Configuration Dalibo**
   ```sql
   -- Activer pg_stat_statements
   CREATE EXTENSION IF NOT EXISTS pg_stat_statements;
   ```

3. **Suivre la formation**
   - Commencez par le module 1 (Fondamentaux)
   - Suivez l'ordre des modules pour une progression logique
   - Pratiquez avec les exercices du module 7

## 📊 Outils recommandés

- **Dalibo** : Analyse de performance PostgreSQL
- **pgAdmin** : Interface graphique pour PostgreSQL
- **psql** : Client en ligne de commande
- **EXPLAIN Visualizer** : Visualisation des plans d'exécution

## 📝 Conventions

- Les exemples SQL sont testés sur PostgreSQL 14+
- Les métriques sont basées sur des environnements de production typiques
- Les temps d'exécution peuvent varier selon la configuration

## 🤝 Contribution

Cette formation est conçue pour être évolutive. N'hésitez pas à proposer des améliorations ou des cas d'usage supplémentaires.

## 📚 Ressources complémentaires

- [Documentation PostgreSQL officielle](https://www.postgresql.org/docs/)
- [Dalibo Documentation](https://dalibo.github.io/pg_qualstats/)
- [PostgreSQL Performance Tuning](https://wiki.postgresql.org/wiki/Performance_Optimization)

