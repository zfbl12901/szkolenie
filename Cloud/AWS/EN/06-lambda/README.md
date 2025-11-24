# 6. AWS Lambda - Serverless Computing

## 🎯 Objectives

- Understand AWS Lambda and its usage
- Create Lambda functions
- Process data with Lambda
- Trigger Lambda from S3
- Integrate Lambda with other services

## 📋 Table of Contents

1. [Introduction to Lambda](#introduction-to-lambda)
2. [Create a Lambda Function](#create-a-lambda-function)
3. [Data Processing](#data-processing)
4. [Triggers](#triggers)
5. [Integration with Other Services](#integration-with-other-services)
6. [Best Practices](#best-practices)

---

## Introduction to Lambda

### What is AWS Lambda?

**AWS Lambda** = Serverless computing service

- **Serverless** : No servers to manage
- **Event-driven** : Triggered by events
- **Auto-scaling** : Automatically adapts
- **Pay-per-use** : Pay only for execution

### Use Cases for Data Analyst

- **File processing** : Transform uploaded files
- **Automated ETL** : Trigger Glue jobs
- **Data validation** : Verify data
- **Notifications** : Alert on events
- **Orchestration** : Coordinate multiple services

### Lambda Free Tier

**Free forever:**
- 1 million requests/month
- 400,000 GB-seconds of compute time/month
- Beyond: usage-based billing

**⚠️ Important:** Very generous for most use cases.

---

## Create a Lambda Function

### Step 1: Access Lambda

1. AWS Console → Search "Lambda"
2. Click on "AWS Lambda"
3. "Create function"

### Step 2: Basic Configuration

**Options:**

1. **Author from scratch** : Create from scratch
2. **Use a blueprint** : Use a template
3. **Browse serverless app repository** : Pre-built applications

**Configuration:**

1. **Function name**: `process-data-file`
2. **Runtime**: Python 3.11 (or other)
3. **Architecture**: x86_64 (default)
4. **Permissions**: Create a new role with basic permissions

### Step 3: Function Code

**Simple example:**

```python
import json

def lambda_handler(event, context):
    """
    Basic Lambda function
    """
    # Processing
    result = {
        'statusCode': 200,
        'body': json.dumps('Hello from Lambda!')
    }
    
    return result
```

**Test the function:**

1. Click on "Test"
2. Create a test event
3. Execute
4. View results

---

## Data Processing

### Example 1: Process a CSV File

```python
import json
import csv
import boto3

s3 = boto3.client('s3')

def lambda_handler(event, context):
    # Get bucket and key from event
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    # Download the file
    response = s3.get_object(Bucket=bucket, Key=key)
    content = response['Body'].read().decode('utf-8')
    
    # Parse CSV
    reader = csv.DictReader(content.splitlines())
    rows = list(reader)
    
    # Process data
    processed = []
    for row in rows:
        processed.append({
            'id': row['id'],
            'name': row['name'].upper(),
            'email': row['email'].lower()
        })
    
    # Upload result
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

### Example 2: Validate Data

```python
import json
import boto3

s3 = boto3.client('s3')

def lambda_handler(event, context):
    bucket = event['bucket']
    key = event['key']
    
    # Download the file
    response = s3.get_object(Bucket=bucket, Key=key)
    data = json.loads(response['Body'].read())
    
    # Validate
    errors = []
    for item in data:
        if 'email' not in item or '@' not in item['email']:
            errors.append(f"Invalid email for id {item.get('id')}")
        if 'age' in item and item['age'] < 0:
            errors.append(f"Invalid age for id {item.get('id')}")
    
    # Upload report
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

### Example 3: Trigger a Glue Job

```python
import boto3

glue = boto3.client('glue')

def lambda_handler(event, context):
    # Name of Glue job to execute
    job_name = 'my-etl-job'
    
    # Trigger the job
    response = glue.start_job_run(
        JobName=job_name
    )
    
    return {
        'statusCode': 200,
        'body': f'Started Glue job: {response["JobRunId"]}'
    }
```

---

## Triggers

### Trigger from S3

**Configuration:**

1. Lambda → Function → "Add trigger"
2. Source: "S3"
3. Bucket: Select the bucket
4. Event type: "All object create events" (or specific)
5. Prefix (optional): `raw/` (only files in this prefix)
6. Suffix (optional): `.csv` (only CSV files)

**Result:** Lambda executes automatically when a file is uploaded.

### Trigger from EventBridge (Schedule)

**Schedule an execution:**

1. Lambda → Function → "Add trigger"
2. Source: "EventBridge (CloudWatch Events)"
3. Rule: Create a new rule
4. Schedule expression: `cron(0 2 * * ? *)` (every day at 2 AM)

**Cron examples:**
- `cron(0 2 * * ? *)` : Every day at 2 AM
- `cron(0 */6 * * ? *)` : Every 6 hours
- `cron(0 0 ? * MON *)` : Every Monday at midnight

### Trigger from API Gateway

**Create a REST API:**

1. API Gateway → "Create API"
2. Type: REST API
3. Create a resource and method
4. Integration: Lambda Function
5. Select the Lambda function

**Result:** HTTP call triggers Lambda.

---

## Integration with Other Services

### Lambda + S3

**Automatic file processing:**

```python
import boto3

s3 = boto3.client('s3')

def lambda_handler(event, context):
    # S3 event
    for record in event['Records']:
        bucket = record['s3']['bucket']['name']
        key = record['s3']['object']['key']
        
        # Process the file
        # ...
```

### Lambda + Glue

**Trigger a Glue job:**

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

**Send a notification:**

```python
import boto3
import json

sns = boto3.client('sns')

def lambda_handler(event, context):
    # Processing...
    
    # Send notification
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

**Orchestrate multiple Lambdas:**

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

## Best Practices

### Performance

1. **Optimize code** to reduce execution time
2. **Use appropriate memory** (128 MB to 10 GB)
3. **Reuse connections** (boto3 clients)
4. **Use layers** for common dependencies

### Costs

1. **Optimize duration** of execution
2. **Choose the right memory** (more memory = faster but more expensive)
3. **Avoid unnecessary timeouts**
4. **Use reservations** if constant usage (not in Free Tier)

### Security

1. **Use IAM roles** for permissions
2. **Don't hardcode** credentials
3. **Use environment variables** for configuration
4. **Enable VPC** if private access needed

### Error Handling

```python
import json
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    try:
        # Processing
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

## Practical Examples

### Example 1: Automated ETL Pipeline

```python
import boto3
import json

s3 = boto3.client('s3')
glue = boto3.client('glue')

def lambda_handler(event, context):
    """
    Triggers a Glue job when a file is uploaded to S3
    """
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    # Check if it's a CSV file
    if not key.endswith('.csv'):
        return {'statusCode': 200, 'body': 'Not a CSV file'}
    
    # Trigger Glue job
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

### Example 2: Validation and Notification

```python
import boto3
import json
import csv

s3 = boto3.client('s3')
sns = boto3.client('sns')

def lambda_handler(event, context):
    bucket = event['bucket']
    key = event['key']
    
    # Download and validate
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

## 📊 Key Takeaways

1. **Lambda = Serverless** : No infrastructure to manage
2. **Free Tier: 1M requests/month** : Very generous
3. **Event-driven** : Triggered by events
4. **Easy integration** : With all AWS services
5. **Pay-per-use** : Pay only for execution

## 🔗 Next Module

Proceed to module [7. Practical Projects](../07-projets/README.md) to create complete projects with AWS.

