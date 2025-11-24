# 3. Obrazy Docker

## 🎯 Cele

- Zrozumieć obrazy Docker
- Pobierać obrazy
- Tworzyć niestandardowe obrazy
- Zarządzać obrazami
- Optymalizować obrazy

## 📋 Spis treści

1. [Wprowadzenie do obrazów](#wprowadzenie-do-obrazów)
2. [Pobierać obrazy](#pobierać-obrazy)
3. [Tworzyć obrazy](#tworzyć-obrazy)
4. [Zarządzać obrazami](#zarządzać-obrazami)
5. [Optymalizacja](#optymalizacja)

---

## Wprowadzenie do obrazów

### Czym jest obraz?

**Obraz** = Szablon tylko do odczytu

- **Szablon** : Do tworzenia kontenerów
- **Warstwy** : Składa się z warstw
- **Niezmienny** : Nie zmienia się
- **Dzielony** : Wiele kontenerów może używać tego samego obrazu

### Struktura obrazu

```
Obraz
├── Warstwa 1 : OS bazowy (Ubuntu)
├── Warstwa 2 : Narzędzia systemowe
├── Warstwa 3 : Python
└── Warstwa 4 : Twoja aplikacja
```

---

## Pobierać obrazy

### Docker Hub

**Docker Hub** = Publiczny rejestr obrazów

- **Obrazy oficjalne** : python, postgres, nginx, itp.
- **Obrazy społecznościowe** : Utworzone przez społeczność
- **Darmowy** : Do użytku publicznego

### Pobierać obraz

```bash
# Pobierać obraz
docker pull python:3.11

# Pobierać najnowszą wersję
docker pull python:latest

# Pobierać konkretną wersję
docker pull python:3.11-slim

# Szukać obrazów
docker search python
```

### Popularne obrazy dla Data Analyst

```bash
# Python
docker pull python:3.11

# Jupyter Notebook
docker pull jupyter/scipy-notebook

# PostgreSQL
docker pull postgres:15

# MySQL
docker pull mysql:8.0

# Redis
docker pull redis:7
```

---

## Tworzyć obrazy

### Z Dockerfile

**Utworzyć Dockerfile:**

```dockerfile
# Dockerfile
FROM python:3.11

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

CMD ["python", "app.py"]
```

**Zbudować obraz:**

```bash
# Zbudować obraz
docker build -t my-app:latest .

# Z konkretnym tagiem
docker build -t my-app:v1.0 .

# Z konkretnego Dockerfile
docker build -f Dockerfile.prod -t my-app:prod .
```

### Commit z kontenera

```bash
# Utworzyć kontener
docker run -it ubuntu bash

# Wprowadzić modyfikacje w kontenerze
apt update
apt install python3

# Utworzyć obraz z kontenera
docker commit container-id my-image:tag
```

---

## Zarządzać obrazami

### Listować obrazy

```bash
# Listować wszystkie obrazy
docker images

# Filtrować po nazwie
docker images python

# Pokazać tylko ID
docker images -q
```

### Usunąć obrazy

```bash
# Usunąć obraz
docker rmi my-image:tag

# Usunąć po ID
docker rmi image-id

# Usunąć nieużywane obrazy
docker image prune

# Usunąć wszystkie obrazy
docker rmi $(docker images -q)
```

### Tagować obrazy

```bash
# Utworzyć tag
docker tag my-image:latest my-image:v1.0

# Tagować dla Docker Hub
docker tag my-image:latest username/my-image:latest
```

### Sprawdzić obraz

```bash
# Pełne informacje
docker inspect my-image

# Historia warstw
docker history my-image

# Rozmiar obrazu
docker images my-image
```

---

## Optymalizacja

### Obrazy lekkie

**Używać obrazów slim:**

```dockerfile
# Zamiast
FROM python:3.11

# Używać
FROM python:3.11-slim
```

**Multi-stage builds:**

```dockerfile
# Stage 1 : Build
FROM python:3.11 as builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user -r requirements.txt

# Stage 2 : Runtime
FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY . .
CMD ["python", "app.py"]
```

### Zmniejszyć rozmiar

**Dobre praktyki:**
1. Używać `.dockerignore`
2. Łączyć polecenia RUN
3. Używać lekkich obrazów bazowych
4. Czyścić cache

---

## 📊 Kluczowe punkty do zapamiętania

1. **Obrazy** to szablony dla kontenerów
2. **Docker Hub** do znajdowania obrazów
3. **Dockerfile** do tworzenia obrazów
4. **Warstwy** umożliwiają dzielenie
5. **Optymalizacja** zmniejsza rozmiar

## 🔗 Następny moduł

Przejdź do modułu [4. Dockerfile](./04-dockerfile/README.md), aby nauczyć się pisać Dockerfile.

