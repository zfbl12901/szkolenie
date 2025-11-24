# 8. Projets pratiques Docker

## 🎯 Objectifs

- Conteneuriser une application Python
- Créer un pipeline de données avec Docker
- Stack complète avec Docker Compose
- Projets pour portfolio

## 📋 Table des matières

1. [Projet 1 : Application Python](#projet-1--application-python)
2. [Projet 2 : Pipeline de données](#projet-2--pipeline-de-données)
3. [Projet 3 : Stack complète](#projet-3--stack-complète)
4. [Projet 4 : Application web](#projet-4--application-web)

---

## Projet 1 : Application Python

### Objectif

Conteneuriser une application Python simple.

### Structure

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

### Construire et exécuter

```bash
# Construire
docker build -t python-app .

# Exécuter
docker run -v $(pwd)/data:/app/data python-app
```

---

## Projet 2 : Pipeline de données

### Objectif

Créer un pipeline ETL avec Docker.

### Structure

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

## Projet 3 : Stack complète

### Objectif

Stack complète avec base de données et application.

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

## Projet 4 : Application web

### Objectif

Application web Flask avec base de données.

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

## 📊 Points clés à retenir

1. **Conteneurisation** : Isoler les applications
2. **Docker Compose** : Orchestrer plusieurs services
3. **Volumes** : Persister les données
4. **Réseaux** : Communication entre services
5. **Portfolio** : Projets démontrables

## 🔗 Ressources

- [Docker Examples](https://github.com/docker/awesome-compose)
- [Docker Documentation](https://docs.docker.com/)

---

**Félicitations !** Vous avez terminé la formation Docker. Vous pouvez maintenant conteneuriser vos applications.

