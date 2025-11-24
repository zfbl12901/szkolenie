# Formation Apache Airflow pour Data Analyst

## 📚 Vue d'ensemble

Cette formation vous guide dans l'apprentissage d'**Apache Airflow** en tant que Data Analyst. Airflow est une plateforme open-source pour orchestrer et automatiser des workflows de données complexes.

## 🎯 Objectifs pédagogiques

- Comprendre Apache Airflow et son rôle dans l'orchestration ETL
- Installer et configurer Airflow
- Créer des DAGs (Directed Acyclic Graphs)
- Utiliser les opérateurs, sensors et hooks
- Orchestrer des pipelines de données complexes
- Intégrer avec des bases de données et services cloud
- Créer des projets pratiques pour votre portfolio

## 💰 Tout est gratuit !

Cette formation utilise uniquement :
- ✅ **Apache Airflow** : Open-source et gratuit
- ✅ **Python** : Langage de programmation gratuit
- ✅ **PostgreSQL/SQLite** : Bases de données gratuites
- ✅ **Documentation officielle** : Guides complets gratuits

**Budget total : 0€**

## 📖 Structure de la formation

### 1. [Prise en main Airflow](./01-getting-started/README.md)
   - Installer Airflow
   - Configuration de base
   - Interface web Airflow
   - Premiers DAGs

### 2. [Concepts fondamentaux](./02-concepts/README.md)
   - DAGs (Directed Acyclic Graphs)
   - Tasks et dépendances
   - Scheduling et triggers
   - Variables et connexions

### 3. [Opérateurs](./03-operators/README.md)
   - Opérateurs Python
   - Opérateurs SQL
   - Opérateurs Bash
   - Opérateurs personnalisés

### 4. [Sensors](./04-sensors/README.md)
   - FileSensor
   - SqlSensor
   - HttpSensor
   - Sensors personnalisés

### 5. [Hooks](./05-hooks/README.md)
   - Hooks de base de données
   - Hooks cloud (AWS, Azure)
   - Hooks HTTP
   - Créer des hooks personnalisés

### 6. [Variables et Connexions](./06-variables-connections/README.md)
   - Gérer les variables
   - Configurer les connexions
   - Sécurité et bonnes pratiques
   - Variables dynamiques

### 7. [Bonnes pratiques](./07-best-practices/README.md)
   - Structure des DAGs
   - Gestion des erreurs
   - Performance et optimisation
   - Tests et débogage

### 8. [Projets pratiques](./08-projets/README.md)
   - Pipeline ETL complet
   - Orchestration de workflows
   - Intégration avec bases de données
   - Projets pour portfolio

## 🚀 Démarrage rapide

### Prérequis

- **Python 3.8+** : Installé sur votre système
- **pip** : Gestionnaire de paquets Python
- **PostgreSQL** (optionnel) : Pour la base de métadonnées

### Installation rapide

```bash
# Créer un environnement virtuel
python -m venv airflow-env

# Activer l'environnement
# Windows
airflow-env\Scripts\activate
# Linux/Mac
source airflow-env/bin/activate

# Installer Airflow
pip install apache-airflow

# Initialiser la base de données
airflow db init

# Créer un utilisateur admin
airflow users create \
    --username admin \
    --firstname Admin \
    --lastname User \
    --role Admin \
    --email admin@example.com

# Démarrer le serveur web
airflow webserver --port 8080

# Dans un autre terminal, démarrer le scheduler
airflow scheduler
```

### Accéder à l'interface web

1. Ouvrir un navigateur
2. Aller sur : `http://localhost:8080`
3. Se connecter avec les credentials créés

## 📊 Cas d'usage pour Data Analyst

- **Orchestration ETL** : Coordonner des pipelines de données
- **Scheduling** : Planifier des tâches récurrentes
- **Monitoring** : Surveiller l'exécution des workflows
- **Gestion d'erreurs** : Retry automatique et alertes
- **Intégration** : Connecter plusieurs outils et services

## ⚠️ Installation sur machine distante

Si vous installez Airflow sur une machine A et souhaitez y accéder depuis une machine B, consultez le guide [Installation et accès distant](./INSTALLATION_REMOTE.md).

## 📚 Ressources gratuites

### Documentation officielle

- **Apache Airflow** : https://airflow.apache.org/docs/
  - Guides complets
  - Tutoriels pas à pas
  - Exemples de code
  - API Reference

- **GitHub Airflow** : https://github.com/apache/airflow
  - Code source
  - Exemples de DAGs
  - Contributions

### Ressources externes

- **YouTube** : Tutoriels Airflow
- **Medium** : Articles et guides
- **Stack Overflow** : Questions et réponses

## 🎓 Certifications (optionnel)

### Apache Airflow (pas de certification officielle)

- **Formation** : Documentation et tutoriels gratuits
- **Durée** : 2-4 semaines
- **Niveau** : Intermédiaire à avancé

## 📝 Conventions

- Tous les exemples utilisent Python 3.8+
- Les DAGs sont testés sur Airflow 2.x
- Les chemins peuvent varier selon le système d'exploitation
- Les ports par défaut peuvent être modifiés

## 🤝 Contribution

Cette formation est conçue pour être évolutive. N'hésitez pas à proposer des améliorations ou des cas d'usage supplémentaires.

## 📚 Ressources complémentaires

- [Documentation Apache Airflow](https://airflow.apache.org/docs/)
- [GitHub Apache Airflow](https://github.com/apache/airflow)
- [Airflow Community](https://airflow.apache.org/community/)
- [Airflow Examples](https://github.com/apache/airflow/tree/main/airflow/example_dags)

