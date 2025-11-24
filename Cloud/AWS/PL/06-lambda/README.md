# 6. AWS Lambda - Serverless Computing

## 🎯 Cele

- Zrozumieć AWS Lambda i jego użycie
- Tworzyć funkcje Lambda
- Przetwarzać dane z Lambda
- Wyzwalać Lambda z S3
- Integrować Lambda z innymi usługami

## 📋 Spis treści

1. [Wprowadzenie do Lambda](#wprowadzenie-do-lambda)
2. [Utworzyć funkcję Lambda](#utworzyć-funkcję-lambda)
3. [Przetwarzanie danych](#przetwarzanie-danych)
4. [Wyzwalacze (Triggers)](#wyzwalacze-triggers)
5. [Integracja z innymi usługami](#integracja-z-innymi-usługami)
6. [Dobre praktyki](#dobre-praktyki)

---

## Wprowadzenie do Lambda

### Czym jest AWS Lambda?

**AWS Lambda** = Usługa obliczeń serverless

- **Serverless** : Brak serwerów do zarządzania
- **Event-driven** : Wyzwalane przez zdarzenia
- **Auto-scaling** : Automatycznie dostosowuje się
- **Pay-per-use** : Płacisz tylko za wykonanie

### Przypadki użycia dla Data Analyst

- **Przetwarzanie plików** : Przekształcać przesłane pliki
- **Automatyzowany ETL** : Wyzwalać joby Glue
- **Walidacja danych** : Weryfikować dane
- **Powiadomienia** : Alertować o zdarzeniach
- **Orkiestracja** : Koordynować wiele usług

### Free Tier Lambda

**Darmowe na zawsze :**
- 1 milion żądań/miesiąc
- 400 000 GB-sekund czasu obliczeń/miesiąc
- Poza tym : rozliczanie według użycia

**⚠️ Ważne :** Bardzo hojne dla większości przypadków użycia.

---

## Utworzyć funkcję Lambda

### Krok 1 : Dostęp do Lambda

1. Konsola AWS → Szukać "Lambda"
2. Kliknąć "AWS Lambda"
3. "Create function"

### Krok 2 : Podstawowa konfiguracja

**Opcje :**

1. **Author from scratch** : Utworzyć od zera
2. **Use a blueprint** : Używać szablonu
3. **Browse serverless app repository** : Aplikacje wstępnie zbudowane

**Konfiguracja :**

1. **Function name** : `process-data-file`
2. **Runtime** : Python 3.11 (lub inny)
3. **Architecture** : x86_64 (domyślnie)
4. **Permissions** : Utworzyć nową rolę z podstawowymi uprawnieniami

### Krok 3 : Kod funkcji

**Prosty przykład :**

```python
import json

def lambda_handler(event, context):
    """
    Podstawowa funkcja Lambda
    """
    # Przetwarzanie
    result = {
        'statusCode': 200,
        'body': json.dumps('Hello from Lambda!')
    }
    
    return result
```

**Testować funkcję :**

1. Kliknąć "Test"
2. Utworzyć zdarzenie testowe
3. Wykonać
4. Zobaczyć wyniki

---

## Przetwarzanie danych

### Przykład 1 : Przetwarzać plik CSV

```python
import json
import csv
import boto3

s3 = boto3.client('s3')

def lambda_handler(event, context):
    # Pobrać bucket i klucz ze zdarzenia
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    # Pobrać plik
    response = s3.get_object(Bucket=bucket, Key=key)
    content = response['Body'].read().decode('utf-8')
    
    # Parsować CSV
    reader = csv.DictReader(content.splitlines())
    rows = list(reader)
    
    # Przetwarzać dane
    processed = []
    for row in rows:
        processed.append({
            'id': row['id'],
            'name': row['name'].upper(),
            'email': row['email'].lower()
        })
    
    # Przesłać wynik
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

### Przykład 2 : Walidować dane

```python
import json
import boto3

s3 = boto3.client('s3')

def lambda_handler(event, context):
    bucket = event['bucket']
    key = event['key']
    
    # Pobrać plik
    response = s3.get_object(Bucket=bucket, Key=key)
    data = json.loads(response['Body'].read())
    
    # Walidować
    errors = []
    for item in data:
        if 'email' not in item or '@' not in item['email']:
            errors.append(f"Invalid email for id {item.get('id')}")
        if 'age' in item and item['age'] < 0:
            errors.append(f"Invalid age for id {item.get('id')}")
    
    # Przesłać raport
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

### Przykład 3 : Wyzwolić job Glue

```python
import boto3

glue = boto3.client('glue')

def lambda_handler(event, context):
    # Nazwa joba Glue do wykonania
    job_name = 'my-etl-job'
    
    # Wyzwolić job
    response = glue.start_job_run(
        JobName=job_name
    )
    
    return {
        'statusCode': 200,
        'body': f'Started Glue job: {response["JobRunId"]}'
    }
```

---

## Wyzwalacze (Triggers)

### Wyzwalać z S3

**Konfiguracja :**

1. Lambda → Function → "Add trigger"
2. Źródło : "S3"
3. Bucket : Wybrać bucket
4. Typ zdarzenia : "All object create events" (lub konkretny)
5. Prefiks (opcjonalne) : `raw/` (tylko pliki w tym prefiksie)
6. Sufiks (opcjonalne) : `.csv` (tylko pliki CSV)

**Wynik :** Lambda wykonuje się automatycznie gdy plik jest przesłany.

### Wyzwalać z EventBridge (harmonogram)

**Zaplanować wykonanie :**

1. Lambda → Function → "Add trigger"
2. Źródło : "EventBridge (CloudWatch Events)"
3. Reguła : Utworzyć nową regułę
4. Wyrażenie harmonogramu : `cron(0 2 * * ? *)` (codziennie o 2h)

**Przykłady cron :**
- `cron(0 2 * * ? *)` : Codziennie o 2h
- `cron(0 */6 * * ? *)` : Co 6 godzin
- `cron(0 0 ? * MON *)` : W każdy poniedziałek o północy

### Wyzwalać z API Gateway

**Utworzyć API REST :**

1. API Gateway → "Create API"
2. Typ : REST API
3. Utworzyć zasób i metodę
4. Integracja : Lambda Function
5. Wybrać funkcję Lambda

**Wynik :** Wywołanie HTTP wyzwala Lambda.

---

## Integracja z innymi usługami

### Lambda + S3

**Automatyczne przetwarzanie plików :**

```python
import boto3

s3 = boto3.client('s3')

def lambda_handler(event, context):
    # Zdarzenie S3
    for record in event['Records']:
        bucket = record['s3']['bucket']['name']
        key = record['s3']['object']['key']
        
        # Przetwarzać plik
        # ...
```

### Lambda + Glue

**Wyzwolić job Glue :**

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

### Lambda + SNS (Powiadomienia)

**Wysłać powiadomienie :**

```python
import boto3
import json

sns = boto3.client('sns')

def lambda_handler(event, context):
    # Przetwarzanie...
    
    # Wysłać powiadomienie
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

**Orkiestrować wiele Lambd :**

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

## Dobre praktyki

### Wydajność

1. **Optymalizować kod** aby zmniejszyć czas wykonania
2. **Używać odpowiedniej pamięci** (128 MB do 10 GB)
3. **Ponownie używać połączeń** (klienci boto3)
4. **Używać layers** dla wspólnych zależności

### Koszty

1. **Optymalizować czas trwania** wykonania
2. **Wybrać odpowiednią pamięć** (więcej pamięci = szybciej ale drożej)
3. **Unikać niepotrzebnych timeoutów**
4. **Używać rezerwacji** jeśli stałe użycie (nie w Free Tier)

### Bezpieczeństwo

1. **Używać ról IAM** dla uprawnień
2. **Nie hardcodować** credentials
3. **Używać zmiennych środowiskowych** do konfiguracji
4. **Włączyć VPC** jeśli potrzeba prywatnego dostępu

### Obsługa błędów

```python
import json
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    try:
        # Przetwarzanie
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

## Przykłady praktyczne

### Przykład 1 : Automatyczny pipeline ETL

```python
import boto3
import json

s3 = boto3.client('s3')
glue = boto3.client('glue')

def lambda_handler(event, context):
    """
    Wyzwala job Glue gdy plik jest przesłany do S3
    """
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    # Sprawdzić czy to plik CSV
    if not key.endswith('.csv'):
        return {'statusCode': 200, 'body': 'Not a CSV file'}
    
    # Wyzwolić job Glue
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

### Przykład 2 : Walidacja i powiadomienie

```python
import boto3
import json
import csv

s3 = boto3.client('s3')
sns = boto3.client('sns')

def lambda_handler(event, context):
    bucket = event['bucket']
    key = event['key']
    
    # Pobrać i walidować
    response = s3.get_object(Bucket=bucket, Key=key)
    content = response['Body'].read().decode('utf-8')
    reader = csv.DictReader(content.splitlines())
    
    errors = []
    for row in reader:
        if not row.get('email') or '@' not in row['email']:
            errors.append(f"Row {row.get('id')}: Invalid email")
    
    # Powiadomienie
    if errors:
        sns.publish(
            TopicArn='arn:aws:sns:region:account:alerts',
            Message=f'Validation failed: {len(errors)} errors found'
        )
    
    return {'statusCode': 200 if not errors else 400}
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **Lambda = Serverless** : Brak infrastruktury do zarządzania
2. **Free Tier : 1M żądań/miesiąc** : Bardzo hojne
3. **Event-driven** : Wyzwalane przez zdarzenia
4. **Łatwa integracja** : Ze wszystkimi usługami AWS
5. **Pay-per-use** : Płacisz tylko za wykonanie

## 🔗 Następny moduł

Przejdź do modułu [7. Projekty praktyczne](../07-projets/README.md), aby tworzyć kompletne projekty z AWS.

