# 2. Kontenery Docker

## 🎯 Cele

- Tworzyć i zarządzać kontenerami
- Zrozumieć cykl życia
- Wykonywać polecenia
- Zarządzać logami
- Konfigurować kontenery

## 📋 Spis treści

1. [Cykl życia kontenera](#cykl-życia-kontenera)
2. [Tworzyć kontenery](#tworzyć-kontenery)
3. [Wykonywać polecenia](#wykonywać-polecenia)
4. [Logi i debugowanie](#logi-i-debugowanie)
5. [Konfiguracja](#konfiguracja)

---

## Cykl życia kontenera

### Stany kontenera

1. **Created** : Kontener utworzony ale nie uruchomiony
2. **Running** : Kontener w trakcie wykonania
3. **Paused** : Kontener wstrzymany
4. **Stopped** : Kontener zatrzymany
5. **Removed** : Kontener usunięty

### Polecenia cyklu życia

```bash
# Utworzyć kontener
docker create --name my-container ubuntu

# Uruchomić kontener
docker start my-container

# Zatrzymać kontener
docker stop my-container

# Uruchomić ponownie kontener
docker restart my-container

# Wstrzymać
docker pause my-container

# Wznowić
docker unpause my-container

# Usunąć kontener
docker rm my-container
```

---

## Tworzyć kontenery

### Tworzyć z docker run

```bash
# Utworzyć i uruchomić kontener
docker run ubuntu echo "Hello"

# Utworzyć bez uruchamiania
docker create --name my-container ubuntu

# Utworzyć z niestandardową nazwą
docker run --name my-app ubuntu
```

### Ważne opcje

```bash
# Tryb interaktywny
docker run -it ubuntu bash

# Tryb odłączony (w tle)
docker run -d nginx

# Udostępnić port
docker run -p 8080:80 nginx

# Zamontować wolumen
docker run -v /host/path:/container/path ubuntu

# Zmienne środowiskowe
docker run -e MY_VAR=value ubuntu

# Nazwa kontenera
docker run --name my-container ubuntu
```

---

## Wykonywać polecenia

### Wykonywać w uruchomionym kontenerze

```bash
# Wykonać polecenie
docker exec my-container ls

# Tryb interaktywny
docker exec -it my-container bash

# Wykonać Python
docker exec -it my-container python
```

### Wykonywać przy starcie

```bash
# Polecenie domyślne
docker run ubuntu echo "Hello"

# Nadpisać polecenie
docker run ubuntu ls -la

# Wykonać skrypt
docker run -v $(pwd):/app ubuntu bash /app/script.sh
```

---

## Logi i debugowanie

### Zobaczyć logi

```bash
# Logi kontenera
docker logs my-container

# Śledzić logi (tail -f)
docker logs -f my-container

# Ostatnie linie
docker logs --tail 100 my-container

# Z timestampem
docker logs -t my-container
```

### Sprawdzić kontener

```bash
# Pełne informacje
docker inspect my-container

# Konkretne informacje
docker inspect --format='{{.State.Status}}' my-container

# Konfiguracja sieci
docker inspect --format='{{.NetworkSettings.IPAddress}}' my-container
```

### Statystyki

```bash
# Statystyki w czasie rzeczywistym
docker stats

# Statystyki kontenera
docker stats my-container

# Statystyki bez streamingu
docker stats --no-stream
```

---

## Konfiguracja

### Zmienne środowiskowe

```bash
# Jedna zmienna
docker run -e MY_VAR=value ubuntu

# Wiele zmiennych
docker run -e VAR1=value1 -e VAR2=value2 ubuntu

# Plik .env
docker run --env-file .env ubuntu
```

### Porty

```bash
# Udostępnić port
docker run -p 8080:80 nginx

# Udostępnić wiele portów
docker run -p 8080:80 -p 3306:3306 my-app

# Udostępnić wszystkie porty
docker run -P nginx
```

### Wolumeny

```bash
# Wolumen nazwany
docker run -v my-volume:/data ubuntu

# Bind mount
docker run -v /host/path:/container/path ubuntu

# Wolumen anonimowy
docker run -v /data ubuntu
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **Cykl życia** : Created → Running → Stopped → Removed
2. **docker run** : Tworzy i uruchamia
3. **docker exec** : Wykonuje w uruchomionym kontenerze
4. **docker logs** : Zobaczyć logi
5. **Konfiguracja** : Zmienne, porty, wolumeny

## 🔗 Następny moduł

Przejdź do modułu [3. Obrazy Docker](./03-images/README.md), aby nauczyć się zarządzania obrazami.

