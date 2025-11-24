# 6. Variables et Connexions

## 🎯 Objectifs

- Gérer les variables Airflow
- Configurer les connexions
- Sécuriser les credentials
- Utiliser des variables dynamiques

## 📋 Table des matières

1. [Variables](#variables)
2. [Connexions](#connexions)
3. [Sécurité](#sécurité)
4. [Bonnes pratiques](#bonnes-pratiques)

---

## Variables

### Créer des variables

**Via CLI :**

```bash
# Créer une variable
airflow variables set my_key "my_value"

# Créer une variable JSON
airflow variables set my_config '{"key": "value"}'

# Supprimer une variable
airflow variables delete my_key

# Lister les variables
airflow variables list
```

**Via interface web :**
1. Admin → Variables
2. Cliquer sur "+"
3. Entrer Key et Value
4. Sauvegarder

### Utiliser des variables

```python
from airflow.models import Variable

# Récupérer une variable
my_value = Variable.get("my_key")

# Avec valeur par défaut
my_value = Variable.get("my_key", default_var="default")

# Variable JSON
config = Variable.get("my_config", deserialize_json=True)
print(config['key'])
```

### Variables dans les templates

```python
from airflow.operators.bash import BashOperator

task = BashOperator(
    task_id='use_var',
    bash_command='echo "Value: {{ var.value.my_key }}"',
    dag=dag,
)
```

---

## Connexions

### Créer une connexion

**Via CLI :**

```bash
# PostgreSQL
airflow connections add 'my_postgres' \
    --conn-type 'postgres' \
    --conn-host 'localhost' \
    --conn-login 'user' \
    --conn-password 'password' \
    --conn-port 5432 \
    --conn-schema 'mydb'

# MySQL
airflow connections add 'my_mysql' \
    --conn-type 'mysql' \
    --conn-host 'localhost' \
    --conn-login 'user' \
    --conn-password 'password' \
    --conn-port 3306

# HTTP
airflow connections add 'my_api' \
    --conn-type 'http' \
    --conn-host 'https://api.example.com'
```

**Via interface web :**
1. Admin → Connections
2. Cliquer sur "+"
3. Remplir les champs
4. Sauvegarder

### Utiliser une connexion

```python
from airflow.hooks.base import BaseHook

# Récupérer une connexion
conn = BaseHook.get_connection('my_postgres')
print(f"Host: {conn.host}")
print(f"Login: {conn.login}")
print(f"Password: {conn.password}")
```

---

## Sécurité

### Masquer les mots de passe

**Utiliser des connexions :**
- Les mots de passe sont chiffrés dans la base
- Ne jamais hardcoder les credentials

**Utiliser des variables :**
- Pour les secrets sensibles
- Utiliser des outils de gestion de secrets (Vault, etc.)

### Bonnes pratiques

1. **Ne jamais commiter** les credentials
2. **Utiliser des connexions** pour les accès
3. **Utiliser des variables** pour la configuration
4. **Chiffrer** les données sensibles
5. **Limiter les accès** aux connexions

---

## Bonnes pratiques

### Organisation des variables

- **Préfixes** : `project_name_key`
- **Groupes** : `db_`, `api_`, `s3_`
- **Documentation** : Documenter l'usage

### Organisation des connexions

- **Noms clairs** : `postgres_prod`, `postgres_dev`
- **Types corrects** : Utiliser le bon type de connexion
- **Tests** : Tester les connexions régulièrement

---

## 📊 Points clés à retenir

1. **Variables** pour la configuration
2. **Connexions** pour les accès
3. **Sécurité** : Ne jamais hardcoder
4. **Organisation** : Préfixes et groupes
5. **Documentation** : Documenter l'usage

## 🔗 Prochain module

Passer au module [7. Bonnes pratiques](../07-best-practices/README.md) pour apprendre les meilleures pratiques.

