# 8. Docker Practical Projects

## 🎯 Objectives

- Containerize a Python application
- Create a data pipeline with Docker
- Complete stack with Docker Compose
- Portfolio projects

## 📋 Table of Contents

1. [Project 1 : Python Application](#project-1--python-application)
2. [Project 2 : Data Pipeline](#project-2--data-pipeline)
3. [Project 3 : Complete Stack](#project-3--complete-stack)
4. [Project 4 : Web Application](#project-4--web-application)

---

## Project 1 : Python Application

### Objective

Containerize a simple Python application.

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

### Build and Run

```bash
# Build
docker build -t python-app .

# Run
docker run -v $(pwd)/data:/app/data python-app
```

---

## Project 2 : Data Pipeline

### Objective

Create an ETL pipeline with Docker.

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

## Project 3 : Complete Stack

### Objective

Complete stack with database and application.

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

## Project 4 : Web Application

### Objective

Flask web application with database.

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

## 📊 Key Takeaways

1. **Containerization** : Isolate applications
2. **Docker Compose** : Orchestrate multiple services
3. **Volumes** : Persist data
4. **Networks** : Communication between services
5. **Portfolio** : Demonstrable projects

## 🔗 Resources

- [Docker Examples](https://github.com/docker/awesome-compose)
- [Docker Documentation](https://docs.docker.com/)

---

**Congratulations!** You have completed the Docker training. You can now containerize your applications.

