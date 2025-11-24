# 6. Zmienne i Połączenia

## 🎯 Cele

- Zarządzać zmiennymi Airflow
- Konfigurować połączenia
- Zabezpieczać credentials
- Używać zmiennych dynamicznych

## 📋 Spis treści

1. [Zmienne](#zmienne)
2. [Połączenia](#połączenia)
3. [Bezpieczeństwo](#bezpieczeństwo)
4. [Dobre praktyki](#dobre-praktyki)

---

## Zmienne

### Tworzyć zmienne

**Przez CLI :**

```bash
# Utworzyć zmienną
airflow variables set my_key "my_value"

# Utworzyć zmienną JSON
airflow variables set my_config '{"key": "value"}'

# Usunąć zmienną
airflow variables delete my_key

# Listować zmienne
airflow variables list
```

**Przez interfejs web :**
1. Admin → Variables
2. Kliknąć "+"
3. Wprowadzić Key i Value
4. Zapisać

### Używać zmiennych

```python
from airflow.models import Variable

# Pobrać zmienną
my_value = Variable.get("my_key")

# Z wartością domyślną
my_value = Variable.get("my_key", default_var="default")

# Zmienna JSON
config = Variable.get("my_config", deserialize_json=True)
print(config['key'])
```

### Zmienne w szablonach

```python
from airflow.operators.bash import BashOperator

task = BashOperator(
    task_id='use_var',
    bash_command='echo "Value: {{ var.value.my_key }}"',
    dag=dag,
)
```

---

## Połączenia

### Utworzyć połączenie

**Przez CLI :**

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

**Przez interfejs web :**
1. Admin → Connections
2. Kliknąć "+"
3. Wypełnić pola
4. Zapisać

### Używać połączenia

```python
from airflow.hooks.base import BaseHook

# Pobrać połączenie
conn = BaseHook.get_connection('my_postgres')
print(f"Host: {conn.host}")
print(f"Login: {conn.login}")
print(f"Password: {conn.password}")
```

---

## Bezpieczeństwo

### Ukrywać hasła

**Używać połączeń :**
- Hasła są szyfrowane w bazie
- Nigdy nie hardkodować credentials

**Używać zmiennych :**
- Dla sekretów wrażliwych
- Używać narzędzi zarządzania sekretami (Vault, itp.)

### Dobre praktyki

1. **Nigdy nie committować** credentials
2. **Używać połączeń** dla dostępu
3. **Używać zmiennych** dla konfiguracji
4. **Szyfrować** dane wrażliwe
5. **Ograniczać dostęp** do połączeń

---

## Dobre praktyki

### Organizacja zmiennych

- **Prefiksy** : `project_name_key`
- **Grupy** : `db_`, `api_`, `s3_`
- **Dokumentacja** : Dokumentować użycie

### Organizacja połączeń

- **Jasne nazwy** : `postgres_prod`, `postgres_dev`
- **Prawidłowe typy** : Używać prawidłowego typu połączenia
- **Testy** : Testować połączenia regularnie

---

## 📊 Kluczowe punkty do zapamiętania

1. **Zmienne** dla konfiguracji
2. **Połączenia** dla dostępu
3. **Bezpieczeństwo** : Nigdy nie hardkodować
4. **Organizacja** : Prefiksy i grupy
5. **Dokumentacja** : Dokumentować użycie

## 🔗 Następny moduł

Przejdź do modułu [7. Dobre praktyki](../07-best-practices/README.md), aby nauczyć się najlepszych praktyk.

