# 8. Projekty praktyczne Docker

## 🎯 Cele

- Konteneryzować aplikację Python
- Tworzyć pipeline danych z Docker
- Kompletny stack z Docker Compose
- Projekty do portfolio

## 📋 Spis treści

1. [Projekt 1 : Aplikacja Python](#projekt-1--aplikacja-python)
2. [Projekt 2 : Pipeline danych](#projekt-2--pipeline-danych)
3. [Projekt 3 : Kompletny stack](#projekt-3--kompletny-stack)
4. [Projekt 4 : Aplikacja web](#projekt-4--aplikacja-web)

---

## Projekt 1 : Aplikacja Python

### Cel

Konteneryzować prostą aplikację Python.

### Struktura

```
python-app/
├── Dockerfile
├── requirements.txt
├── app.py
└── data/
    └── data.csv
```

### Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "app.py"]
```

### app.py

```python
import pandas as pd

def main():
    df = pd.read_csv('data/data.csv')
    print(f"Loaded {len(df)} rows")
    print(df.head())

if __name__ == '__main__':
    main()
```

### Zbudować i uruchomić

```bash
# Zbudować
docker build -t python-app .

# Uruchomić
docker run -v $(pwd)/data:/app/data python-app
```

---

## Projekt 2 : Pipeline danych

### Cel

Utworzyć pipeline ETL z Docker.

### Struktura

```
etl-pipeline/
├── docker-compose.yml
├── extract/
│   ├── Dockerfile
│   └── extract.py
├── transform/
│   ├── Dockerfile
│   └── transform.py
└── load/
    ├── Dockerfile
    └── load.py
```

### docker-compose.yml

```yaml
version: '3.8'

services:
  extract:
    build: ./extract
    volumes:
      - ./data:/data
  
  transform:
    build: ./transform
    depends_on:
      - extract
    volumes:
      - ./data:/data
  
  load:
    build: ./load
    depends_on:
      - transform
    volumes:
      - ./data:/data
```

---

## Projekt 3 : Kompletny stack

### Cel

Kompletny stack z bazą danych i aplikacją.

### docker-compose.yml

```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8080:8000"
    environment:
      - DATABASE_URL=postgresql://user:password@db:5432/mydb
    depends_on:
      - db
    volumes:
      - ./src:/app
  
  db:
    image: postgres:15
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - db-data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
  
  redis:
    image: redis:7
    ports:
      - "6379:6379"

volumes:
  db-data:
```

---

## Projekt 4 : Aplikacja web

### Cel

Aplikacja web Flask z bazą danych.

### Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["flask", "run", "--host", "0.0.0.0"]
```

### docker-compose.yml

```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "5000:5000"
    environment:
      - FLASK_ENV=development
      - DATABASE_URL=postgresql://user:password@db:5432/appdb
    depends_on:
      - db
    volumes:
      - .:/app
  
  db:
    image: postgres:15
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **Konteneryzacja** : Izolować aplikacje
2. **Docker Compose** : Orkiestrować wiele usług
3. **Wolumeny** : Trwać dane
4. **Sieci** : Komunikacja między usługami
5. **Portfolio** : Projekty demonstrowalne

## 🔗 Zasoby

- [Przykłady Docker](https://github.com/docker/awesome-compose)
- [Dokumentacja Docker](https://docs.docker.com/)

---

**Gratulacje !** Ukończyłeś szkolenie Docker. Możesz teraz konteneryzować swoje aplikacje.

