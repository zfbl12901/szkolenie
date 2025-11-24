# 6. AWS Lambda - Serverless Computing

## 🎯 Objectifs

- Comprendre AWS Lambda et son utilisation
- Créer des fonctions Lambda
- Traiter des données avec Lambda
- Déclencher Lambda depuis S3
- Intégrer Lambda avec d'autres services

## 📋 Table des matières

1. [Introduction à Lambda](#introduction-à-lambda)
2. [Créer une fonction Lambda](#créer-une-fonction-lambda)
3. [Traitement de données](#traitement-de-données)
4. [Déclencheurs (Triggers)](#déclencheurs-triggers)
5. [Intégration avec autres services](#intégration-avec-autres-services)
6. [Bonnes pratiques](#bonnes-pratiques)

---

## Introduction à Lambda

### Qu'est-ce qu'AWS Lambda ?

**AWS Lambda** = Service de calcul serverless

- **Serverless** : Pas de serveurs à gérer
- **Event-driven** : Déclenché par événements
- **Auto-scaling** : S'adapte automatiquement
- **Pay-per-use** : Payez seulement l'exécution

### Cas d'usage pour Data Analyst

- **Traitement de fichiers** : Transformer fichiers uploadés
- **ETL automatisé** : Déclencher des jobs Glue
- **Validation de données** : Vérifier les données
- **Notifications** : Alerter sur événements
- **Orchestration** : Coordonner plusieurs services

### Free Tier Lambda

**Gratuit à vie :**
- 1 million de requêtes/mois
- 400 000 Go-secondes de temps de calcul/mois
- Au-delà : facturation à l'usage

**⚠️ Important :** Très généreux pour la plupart des cas d'usage.

---

## Créer une fonction Lambda

### Étape 1 : Accéder à Lambda

1. Console AWS → Rechercher "Lambda"
2. Cliquer sur "AWS Lambda"
3. "Create function"

### Étape 2 : Configuration de base

**Options :**

1. **Author from scratch** : Créer depuis zéro
2. **Use a blueprint** : Utiliser un template
3. **Browse serverless app repository** : Applications pré-construites

**Configuration :**

1. **Function name** : `process-data-file`
2. **Runtime** : Python 3.11 (ou autre)
3. **Architecture** : x86_64 (par défaut)
4. **Permissions** : Créer un nouveau rôle avec permissions de base

### Étape 3 : Code de la fonction

**Exemple simple :**

```python
import json

def lambda_handler(event, context):
    """
    Fonction Lambda de base
    """
    # Traitement
    result = {
        'statusCode': 200,
        'body': json.dumps('Hello from Lambda!')
    }
    
    return result
```

**Tester la fonction :**

1. Cliquer sur "Test"
2. Créer un événement de test
3. Exécuter
4. Voir les résultats

---

## Traitement de données

### Exemple 1 : Traiter un fichier CSV

```python
import json
import csv
import boto3

s3 = boto3.client('s3')

def lambda_handler(event, context):
    # Récupérer le bucket et la clé depuis l'événement
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    # Télécharger le fichier
    response = s3.get_object(Bucket=bucket, Key=key)
    content = response['Body'].read().decode('utf-8')
    
    # Parser le CSV
    reader = csv.DictReader(content.splitlines())
    rows = list(reader)
    
    # Traiter les données
    processed = []
    for row in rows:
        processed.append({
            'id': row['id'],
            'name': row['name'].upper(),
            'email': row['email'].lower()
        })
    
    # Uploader le résultat
    output_key = key.replace('raw/', 'processed/')
    s3.put_object(
        Bucket=bucket,
        Key=output_key,
        Body=json.dumps(processed)
    )
    
    return {
        'statusCode': 200,
        'body': f'Processed {len(processed)} rows'
    }
```

### Exemple 2 : Valider des données

```python
import json
import boto3

s3 = boto3.client('s3')

def lambda_handler(event, context):
    bucket = event['bucket']
    key = event['key']
    
    # Télécharger le fichier
    response = s3.get_object(Bucket=bucket, Key=key)
    data = json.loads(response['Body'].read())
    
    # Valider
    errors = []
    for item in data:
        if 'email' not in item or '@' not in item['email']:
            errors.append(f"Invalid email for id {item.get('id')}")
        if 'age' in item and item['age'] < 0:
            errors.append(f"Invalid age for id {item.get('id')}")
    
    # Uploader le rapport
    if errors:
        s3.put_object(
            Bucket=bucket,
            Key=f'validation-errors/{key}',
            Body=json.dumps(errors)
        )
        return {
            'statusCode': 400,
            'body': f'Found {len(errors)} validation errors'
        }
    
    return {
        'statusCode': 200,
        'body': 'Validation passed'
    }
```

### Exemple 3 : Déclencher un job Glue

```python
import boto3

glue = boto3.client('glue')

def lambda_handler(event, context):
    # Nom du job Glue à exécuter
    job_name = 'my-etl-job'
    
    # Déclencher le job
    response = glue.start_job_run(
        JobName=job_name
    )
    
    return {
        'statusCode': 200,
        'body': f'Started Glue job: {response["JobRunId"]}'
    }
```

---

## Déclencheurs (Triggers)

### Déclencher depuis S3

**Configuration :**

1. Lambda → Function → "Add trigger"
2. Source : "S3"
3. Bucket : Sélectionner le bucket
4. Event type : "All object create events" (ou spécifique)
5. Prefix (optionnel) : `raw/` (seulement fichiers dans ce préfixe)
6. Suffix (optionnel) : `.csv` (seulement fichiers CSV)

**Résultat :** Lambda s'exécute automatiquement quand un fichier est uploadé.

### Déclencher depuis EventBridge (schedule)

**Planifier une exécution :**

1. Lambda → Function → "Add trigger"
2. Source : "EventBridge (CloudWatch Events)"
3. Rule : Créer une nouvelle règle
4. Schedule expression : `cron(0 2 * * ? *)` (tous les jours à 2h)

**Exemple de cron :**
- `cron(0 2 * * ? *)` : Tous les jours à 2h
- `cron(0 */6 * * ? *)` : Toutes les 6 heures
- `cron(0 0 ? * MON *)` : Tous les lundis à minuit

### Déclencher depuis API Gateway

**Créer une API REST :**

1. API Gateway → "Create API"
2. Type : REST API
3. Créer une ressource et méthode
4. Intégration : Lambda Function
5. Sélectionner la fonction Lambda

**Résultat :** Appel HTTP déclenche Lambda.

---

## Intégration avec autres services

### Lambda + S3

**Traitement automatique de fichiers :**

```python
import boto3

s3 = boto3.client('s3')

def lambda_handler(event, context):
    # Événement S3
    for record in event['Records']:
        bucket = record['s3']['bucket']['name']
        key = record['s3']['object']['key']
        
        # Traiter le fichier
        # ...
```

### Lambda + Glue

**Déclencher un job Glue :**

```python
import boto3

glue = boto3.client('glue')

def lambda_handler(event, context):
    response = glue.start_job_run(
        JobName='my-etl-job',
        Arguments={
            '--input-path': 's3://bucket/raw/',
            '--output-path': 's3://bucket/processed/'
        }
    )
    return response
```

### Lambda + SNS (Notifications)

**Envoyer une notification :**

```python
import boto3
import json

sns = boto3.client('sns')

def lambda_handler(event, context):
    # Traitement...
    
    # Envoyer notification
    sns.publish(
        TopicArn='arn:aws:sns:region:account:topic',
        Message=json.dumps({
            'status': 'success',
            'message': 'Data processing completed'
        })
    )
    
    return {'statusCode': 200}
```

### Lambda + Step Functions

**Orchestrer plusieurs Lambdas :**

```json
{
  "Comment": "ETL Pipeline",
  "StartAt": "ProcessData",
  "States": {
    "ProcessData": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:region:account:function:process-data",
      "Next": "ValidateData"
    },
    "ValidateData": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:region:account:function:validate-data",
      "End": true
    }
  }
}
```

---

## Bonnes pratiques

### Performance

1. **Optimiser le code** pour réduire le temps d'exécution
2. **Utiliser les bonnes mémoires** (128 MB à 10 GB)
3. **Réutiliser les connexions** (boto3 clients)
4. **Utiliser des layers** pour dépendances communes

### Coûts

1. **Optimiser la durée** d'exécution
2. **Choisir la bonne mémoire** (plus de mémoire = plus rapide mais plus cher)
3. **Éviter les timeouts** inutiles
4. **Utiliser des réservations** si usage constant (pas dans Free Tier)

### Sécurité

1. **Utiliser IAM roles** pour permissions
2. **Ne pas hardcoder** les credentials
3. **Utiliser des variables d'environnement** pour configuration
4. **Activer VPC** si besoin d'accès privé

### Gestion d'erreurs

```python
import json
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    try:
        # Traitement
        result = process_data(event)
        return {
            'statusCode': 200,
            'body': json.dumps(result)
        }
    except Exception as e:
        logger.error(f'Error: {str(e)}')
        return {
            'statusCode': 500,
            'body': json.dumps({'error': str(e)})
        }
```

---

## Exemples pratiques

### Exemple 1 : Pipeline ETL automatique

```python
import boto3
import json

s3 = boto3.client('s3')
glue = boto3.client('glue')

def lambda_handler(event, context):
    """
    Déclenche un job Glue quand un fichier est uploadé dans S3
    """
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    # Vérifier que c'est un fichier CSV
    if not key.endswith('.csv'):
        return {'statusCode': 200, 'body': 'Not a CSV file'}
    
    # Déclencher le job Glue
    response = glue.start_job_run(
        JobName='csv-to-parquet-job',
        Arguments={
            '--input-path': f's3://{bucket}/{key}',
            '--output-path': f's3://{bucket}/processed/'
        }
    )
    
    return {
        'statusCode': 200,
        'body': f'Started Glue job: {response["JobRunId"]}'
    }
```

### Exemple 2 : Validation et notification

```python
import boto3
import json
import csv

s3 = boto3.client('s3')
sns = boto3.client('sns')

def lambda_handler(event, context):
    bucket = event['bucket']
    key = event['key']
    
    # Télécharger et valider
    response = s3.get_object(Bucket=bucket, Key=key)
    content = response['Body'].read().decode('utf-8')
    reader = csv.DictReader(content.splitlines())
    
    errors = []
    for row in reader:
        if not row.get('email') or '@' not in row['email']:
            errors.append(f"Row {row.get('id')}: Invalid email")
    
    # Notification
    if errors:
        sns.publish(
            TopicArn='arn:aws:sns:region:account:alerts',
            Message=f'Validation failed: {len(errors)} errors found'
        )
    
    return {'statusCode': 200 if not errors else 400}
```

---

## 📊 Points clés à retenir

1. **Lambda = Serverless** : Pas d'infrastructure à gérer
2. **Free Tier : 1M requêtes/mois** : Très généreux
3. **Event-driven** : Déclenché par événements
4. **Intégration facile** : Avec tous les services AWS
5. **Pay-per-use** : Payez seulement l'exécution

## 🔗 Prochain module

Passer au module [7. Projets pratiques](../07-projets/README.md) pour créer des projets complets avec AWS.

