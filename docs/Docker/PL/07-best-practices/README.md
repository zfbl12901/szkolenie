# 7. Dobre praktyki Docker

## 🎯 Cele

- Zabezpieczać kontenery
- Optymalizować wydajność
- Organizować projekty
- Konserwować obrazy
- Zarządzać zasobami

## 📋 Spis treści

1. [Bezpieczeństwo](#bezpieczeństwo)
2. [Wydajność](#wydajność)
3. [Organizacja](#organizacja)
4. [Konserwacja](#konserwacja)
5. [Zasoby](#zasoby)

---

## Bezpieczeństwo

### Używać obrazów oficjalnych

```dockerfile
# Dobre
FROM python:3.11-slim

# Unikać
FROM random-user/python:latest
```

### Nie używać root

```dockerfile
# Utworzyć użytkownika nie-root
RUN useradd -m appuser
USER appuser
```

### Ograniczać uprawnienia

```bash
# Nie używać --privileged
docker run --privileged my-container  # Unikać

# Używać konkretnych capabilities jeśli potrzebne
docker run --cap-add NET_ADMIN my-container
```

### Sekrety

```yaml
# docker-compose.yml
services:
  app:
    secrets:
      - db_password

secrets:
  db_password:
    file: ./secrets/password.txt
```

---

## Wydajność

### Używać .dockerignore

```
__pycache__
*.pyc
.git
.env
node_modules
*.log
```

### Optymalizować warstwy

```dockerfile
# Złe
RUN apt update
RUN apt install -y python3
RUN apt clean

# Dobre
RUN apt update && \
    apt install -y python3 && \
    apt clean
```

### Cache buildów

**Kolejność instrukcji:**

```dockerfile
# Najpierw zależności (zmieniają się rzadko)
COPY requirements.txt .
RUN pip install -r requirements.txt

# Potem kod (zmienia się często)
COPY . .
```

---

## Organizacja

### Struktura projektu

```
my-project/
├── Dockerfile
├── docker-compose.yml
├── .dockerignore
├── requirements.txt
├── src/
│   └── app.py
└── data/
    └── data.csv
```

### Tagi obrazów

```bash
# Tagi semantyczne
docker build -t my-app:1.0.0 .
docker build -t my-app:latest .

# Tagi dla środowiska
docker build -t my-app:dev .
docker build -t my-app:prod .
```

---

## Konserwacja

### Czyścić zasoby

```bash
# Usunąć zatrzymane kontenery
docker container prune

# Usunąć nieużywane obrazy
docker image prune

# Usunąć nieużywane wolumeny
docker volume prune

# Wyczyścić wszystko
docker system prune -a
```

### Aktualizować obrazy

```bash
# Aktualizować obraz
docker pull python:3.11

# Przebudować
docker-compose build --no-cache
docker-compose up
```

---

## Zasoby

### Ograniczać zasoby

```yaml
# docker-compose.yml
services:
  app:
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **Bezpieczeństwo** : Obrazy oficjalne, nie-root
2. **Wydajność** : Optymalizować warstwy, cache
3. **Organizacja** : Czytelna struktura, tagi
4. **Konserwacja** : Czyścić regularnie
5. **Zasoby** : Ograniczać użycie

## 🔗 Następny moduł

Przejdź do modułu [8. Projekty praktyczne](./08-projets/README.md), aby tworzyć kompletne projekty.

