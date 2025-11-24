# 5. Docker Compose

## 🎯 Cele

- Zrozumieć Docker Compose
- Orkiestrować wiele kontenerów
- Tworzyć pliki docker-compose.yml
- Zarządzać usługami i sieciami
- Używać zmiennych środowiskowych

## 📋 Spis treści

1. [Wprowadzenie do Docker Compose](#wprowadzenie-do-docker-compose)
2. [Plik docker-compose.yml](#plik-docker-composeyml)
3. [Usługi](#usługi)
4. [Sieci i Wolumeny](#sieci-i-wolumeny)
5. [Polecenia](#polecenia)

---

## Wprowadzenie do Docker Compose

### Czym jest Docker Compose?

**Docker Compose** = Narzędzie do orkiestracji wielu kontenerów

- **Multi-kontenery** : Zarządza wieloma kontenerami
- **Konfiguracja** : Prosty plik YAML
- **Orkiestracja** : Uruchamia/zatrzymuje wszystkie usługi
- **Sieci** : Automatycznie tworzy sieci

### Dlaczego Docker Compose?

- **Prostota** : Jeden plik dla wszystkiego
- **Reprodukowalność** : To samo środowisko wszędzie
- **Rozwój** : Kompletny stack lokalny
- **Produkcja** : Uproszczone wdrażanie

---

## Plik docker-compose.yml

### Podstawowa struktura

```yaml
version: '3.8'

services:
  web:
    image: nginx
    ports:
      - "8080:80"
  
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: password
```

### Kompletny przykład

```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8080:8000"
    environment:
      - DATABASE_URL=postgresql://db:5432/mydb
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

volumes:
  db-data:
```

---

## Usługi

### Zdefiniować usługę

```yaml
services:
  app:
    image: python:3.11
    command: python app.py
    working_dir: /app
    volumes:
      - .:/app
```

### Build vs Image

```yaml
# Używać istniejącego obrazu
services:
  web:
    image: nginx:latest

# Budować z Dockerfile
services:
  app:
    build: .
    # lub
    build:
      context: .
      dockerfile: Dockerfile.prod
```

### Zależności

```yaml
services:
  app:
    depends_on:
      - db
      - redis
  
  db:
    image: postgres
  
  redis:
    image: redis
```

---

## Sieci i Wolumeny

### Sieci

```yaml
services:
  app:
    networks:
      - frontend
      - backend
  
  db:
    networks:
      - backend

networks:
  frontend:
  backend:
```

### Wolumeny

```yaml
services:
  db:
    volumes:
      - db-data:/var/lib/postgresql/data
      - ./backup:/backup

volumes:
  db-data:
```

---

## Polecenia

### Uruchomić usługi

```bash
# Uruchomić wszystkie usługi
docker-compose up

# W tle
docker-compose up -d

# Przebudować obrazy
docker-compose up --build
```

### Zatrzymać usługi

```bash
# Zatrzymać usługi
docker-compose stop

# Zatrzymać i usunąć
docker-compose down

# Usunąć z wolumenami
docker-compose down -v
```

### Zarządzanie usługami

```bash
# Zobaczyć uruchomione usługi
docker-compose ps

# Zobaczyć logi
docker-compose logs

# Logi usługi
docker-compose logs web

# Wykonać polecenie
docker-compose exec web bash

# Uruchomić ponownie usługę
docker-compose restart web
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **Docker Compose** orkiestruje wiele kontenerów
2. **docker-compose.yml** definiuje konfigurację
3. **Usługi** to kontenery
4. **Sieci i Wolumeny** dla komunikacji i danych
5. **Polecenia** : up, down, logs, exec

## 🔗 Następny moduł

Przejdź do modułu [6. Wolumeny i Sieci](./06-volumes-networks/README.md), aby pogłębić.

