# 6. Variables and Connections

## 🎯 Objectives

- Manage Airflow variables
- Configure connections
- Secure credentials
- Use dynamic variables

## 📋 Table of Contents

1. [Variables](#variables)
2. [Connections](#connections)
3. [Security](#security)
4. [Best Practices](#best-practices)

---

## Variables

### Create Variables

**Via CLI:**

```bash
# Create a variable
airflow variables set my_key "my_value"

# Create a JSON variable
airflow variables set my_config '{"key": "value"}'

# Delete a variable
airflow variables delete my_key

# List variables
airflow variables list
```

**Via web interface:**
1. Admin → Variables
2. Click on "+"
3. Enter Key and Value
4. Save

### Use Variables

```python
from airflow.models import Variable

# Get a variable
my_value = Variable.get("my_key")

# With default value
my_value = Variable.get("my_key", default_var="default")

# JSON variable
config = Variable.get("my_config", deserialize_json=True)
print(config['key'])
```

### Variables in Templates

```python
from airflow.operators.bash import BashOperator

task = BashOperator(
    task_id='use_var',
    bash_command='echo "Value: {{ var.value.my_key }}"',
    dag=dag,
)
```

---

## Connections

### Create a Connection

**Via CLI:**

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

**Via web interface:**
1. Admin → Connections
2. Click on "+"
3. Fill in fields
4. Save

### Use a Connection

```python
from airflow.hooks.base import BaseHook

# Get a connection
conn = BaseHook.get_connection('my_postgres')
print(f"Host: {conn.host}")
print(f"Login: {conn.login}")
print(f"Password: {conn.password}")
```

---

## Security

### Hide Passwords

**Use connections:**
- Passwords are encrypted in database
- Never hardcode credentials

**Use variables:**
- For sensitive secrets
- Use secret management tools (Vault, etc.)

### Best Practices

1. **Never commit** credentials
2. **Use connections** for access
3. **Use variables** for configuration
4. **Encrypt** sensitive data
5. **Limit access** to connections

---

## Best Practices

### Variable Organization

- **Prefixes** : `project_name_key`
- **Groups** : `db_`, `api_`, `s3_`
- **Documentation** : Document usage

### Connection Organization

- **Clear names** : `postgres_prod`, `postgres_dev`
- **Correct types** : Use correct connection type
- **Testing** : Test connections regularly

---

## 📊 Key Takeaways

1. **Variables** for configuration
2. **Connections** for access
3. **Security** : Never hardcode
4. **Organization** : Prefixes and groups
5. **Documentation** : Document usage

## 🔗 Next Module

Proceed to module [7. Best Practices](../07-best-practices/README.md) to learn best practices.

