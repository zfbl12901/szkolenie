# 4. Dockerfile

## 🎯 Cele

- Pisać Dockerfile
- Zrozumieć instrukcje
- Optymalizować Dockerfile
- Używać multi-stage builds
- Dobre praktyki

## 📋 Spis treści

1. [Wprowadzenie do Dockerfile](#wprowadzenie-do-dockerfile)
2. [Podstawowe instrukcje](#podstawowe-instrukcje)
3. [Kompletny Dockerfile](#kompletny-dockerfile)
4. [Optymalizacja](#optymalizacja)
5. [Multi-stage builds](#multi-stage-builds)

---

## Wprowadzenie do Dockerfile

### Czym jest Dockerfile?

**Dockerfile** = Instrukcje do budowania obrazu

- **Tekst** : Prosty plik tekstowy
- **Instrukcje** : Każda linia to instrukcja
- **Automatyzacja** : Automatyzuje tworzenie obrazów
- **Wersjonowany** : Może być wersjonowany z Git

### Podstawowa struktura

```dockerfile
# Komentarz
FROM base-image
RUN command
COPY source destination
CMD ["executable", "param"]
```

---

## Podstawowe instrukcje

### FROM

**Definiuje obraz bazowy:**

```dockerfile
FROM python:3.11
FROM ubuntu:22.04
FROM alpine:latest
```

### WORKDIR

**Definiuje katalog roboczy:**

```dockerfile
WORKDIR /app
WORKDIR /usr/src/app
```

### COPY / ADD

**Kopiować pliki:**

```dockerfile
# COPY (zalecane)
COPY requirements.txt .
COPY . .

# ADD (z automatyczną ekstrakcją)
ADD archive.tar.gz /app
```

### RUN

**Wykonywać polecenia:**

```dockerfile
RUN apt update
RUN pip install -r requirements.txt

# Łączyć aby zmniejszyć warstwy
RUN apt update && \
    apt install -y python3 && \
    apt clean
```

### CMD / ENTRYPOINT

**Polecenie domyślne:**

```dockerfile
# CMD (może być nadpisane)
CMD ["python", "app.py"]

# ENTRYPOINT (nie może być nadpisane)
ENTRYPOINT ["python"]
CMD ["app.py"]
```

### ENV

**Zmienne środowiskowe:**

```dockerfile
ENV PYTHONUNBUFFERED=1
ENV APP_ENV=production
```

### EXPOSE

**Udostępnić porty:**

```dockerfile
EXPOSE 8080
EXPOSE 3306
```

---

## Kompletny Dockerfile

### Przykład 1 : Aplikacja Python

```dockerfile
# Obraz bazowy
FROM python:3.11-slim

# Katalog roboczy
WORKDIR /app

# Zmienne środowiskowe
ENV PYTHONUNBUFFERED=1

# Kopiować requirements
COPY requirements.txt .

# Zainstalować zależności
RUN pip install --no-cache-dir -r requirements.txt

# Kopiować kod
COPY . .

# Udostępnić port
EXPOSE 8000

# Polecenie domyślne
CMD ["python", "app.py"]
```

### Przykład 2 : Aplikacja z danymi

```dockerfile
FROM python:3.11

WORKDIR /app

# Zainstalować zależności systemowe
RUN apt update && \
    apt install -y postgresql-client && \
    apt clean

# Zainstalować zależności Python
COPY requirements.txt .
RUN pip install -r requirements.txt

# Kopiować aplikację
COPY . .

# Utworzyć wolumen dla danych
VOLUME ["/app/data"]

# Udostępnić port
EXPOSE 8080

# Polecenie
CMD ["python", "main.py"]
```

---

## Optymalizacja

### .dockerignore

**Utworzyć plik `.dockerignore`:**

```
__pycache__
*.pyc
.git
.env
node_modules
*.log
.DS_Store
```

### Zmniejszyć warstwy

**Złe:**
```dockerfile
RUN apt update
RUN apt install -y python3
RUN apt install -y pip
RUN apt clean
```

**Dobre:**
```dockerfile
RUN apt update && \
    apt install -y python3 pip && \
    apt clean
```

### Kolejność instrukcji

**Umieścić instrukcje które zmieniają się rzadko najpierw:**

```dockerfile
# Najpierw zależności (zmieniają się rzadko)
COPY requirements.txt .
RUN pip install -r requirements.txt

# Potem kod (zmienia się często)
COPY . .
```

---

## Multi-stage builds

### Dlaczego multi-stage?

- **Zmniejszyć rozmiar** : Obraz końcowy mniejszy
- **Bezpieczeństwo** : Wykluczyć narzędzia build
- **Wydajność** : Optymalizować buildy

### Przykład

```dockerfile
# Stage 1 : Build
FROM python:3.11 as builder
WORKDIR /build
COPY requirements.txt .
RUN pip install --user -r requirements.txt

# Stage 2 : Runtime
FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY . .
ENV PATH=/root/.local/bin:$PATH
CMD ["python", "app.py"]
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **Dockerfile** automatyzuje tworzenie obrazów
2. **Instrukcje** : FROM, RUN, COPY, CMD
3. **Optymalizacja** : Zmniejszyć warstwy
4. **Multi-stage** : Lżejsze obrazy
5. **.dockerignore** : Wykluczyć pliki

## 🔗 Następny moduł

Przejdź do modułu [5. Docker Compose](./05-docker-compose/README.md), aby orkiestrować wiele kontenerów.

