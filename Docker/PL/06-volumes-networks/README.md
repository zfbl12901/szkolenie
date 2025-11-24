# 6. Wolumeny i Sieci Docker

## 🎯 Cele

- Zrozumieć wolumeny Docker
- Zarządzać trwałością danych
- Tworzyć i zarządzać sieciami
- Dzielić dane między kontenerami
- Konfigurować komunikację sieciową

## 📋 Spis treści

1. [Wolumeny](#wolumeny)
2. [Bind Mounts](#bind-mounts)
3. [Sieci](#sieci)
4. [Komunikacja między kontenerami](#komunikacja-między-kontenerami)
5. [Przykłady praktyczne](#przykłady-praktyczne)

---

## Wolumeny

### Czym jest wolumen?

**Wolumen** = Trwały magazyn dla danych

- **Trwały** : Przetrwa usunięcie kontenera
- **Zarządzany przez Docker** : Przechowywany w `/var/lib/docker/volumes`
- **Dzielony** : Wiele kontenerów może go używać
- **Wydajny** : Szybszy niż bind mounts

### Utworzyć wolumen

```bash
# Utworzyć wolumen
docker volume create my-volume

# Listować wolumeny
docker volume ls

# Sprawdzić wolumen
docker volume inspect my-volume

# Usunąć wolumen
docker volume rm my-volume
```

### Używać wolumenu

```bash
# Wolumen nazwany
docker run -v my-volume:/data ubuntu

# Wolumen anonimowy
docker run -v /data ubuntu

# W docker-compose.yml
volumes:
  - my-volume:/data
```

---

## Bind Mounts

### Czym jest bind mount?

**Bind Mount** = Bezpośrednie połączenie z katalogiem hosta

- **Bezpośredni** : Bezpośredni dostęp do plików hosta
- **Rozwój** : Idealny do rozwoju
- **Wydajność** : Zależy od systemu plików hosta

### Używać bind mount

```bash
# Bind mount
docker run -v /host/path:/container/path ubuntu

# Z docker-compose.yml
volumes:
  - ./data:/app/data
  - /absolute/path:/container/path
```

### Różnice : Wolumen vs Bind Mount

**Wolumen:**
- Zarządzany przez Docker
- Lepsza wydajność
- Przenośny
- Zalecany do produkcji

**Bind Mount:**
- Bezpośrednie połączenie
- Bezpośredni dostęp
- Zależy od systemu hosta
- Zalecany do rozwoju

---

## Sieci

### Typy sieci

1. **Bridge** : Sieć domyślna (izolacja)
2. **Host** : Używa sieci hosta
3. **None** : Brak sieci
4. **Overlay** : Dla Docker Swarm

### Utworzyć sieć

```bash
# Utworzyć sieć
docker network create my-network

# Listować sieci
docker network ls

# Sprawdzić sieć
docker network inspect my-network

# Usunąć sieć
docker network rm my-network
```

### Połączyć kontener

```bash
# Połączyć przy starcie
docker run --network my-network ubuntu

# Połączyć istniejący kontener
docker network connect my-network container-id

# Rozłączyć
docker network disconnect my-network container-id
```

---

## Komunikacja między kontenerami

### Ta sama sieć

```bash
# Utworzyć sieć
docker network create app-network

# Kontener 1
docker run --name app --network app-network my-app

# Kontener 2 (może komunikować się z app)
docker run --name db --network app-network postgres
```

### Z Docker Compose

```yaml
services:
  app:
    networks:
      - app-network
  
  db:
    networks:
      - app-network

networks:
  app-network:
```

### Rozwiązanie DNS

**Kontenery mogą znajdować się po nazwie:**

```python
# W kontenerze app
import psycopg2
conn = psycopg2.connect(
    host="db",  # Nazwa usługi
    database="mydb"
)
```

---

## Przykłady praktyczne

### Przykład 1 : Baza danych z wolumenem

```yaml
version: '3.8'

services:
  db:
    image: postgres:15
    volumes:
      - db-data:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: mydb

volumes:
  db-data:
```

### Przykład 2 : Aplikacja z bind mount

```yaml
version: '3.8'

services:
  app:
    build: .
    volumes:
      - ./src:/app/src  # Rozwój
    networks:
      - app-network

networks:
  app-network:
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **Wolumeny** dla trwałości zarządzanej przez Docker
2. **Bind Mounts** dla bezpośredniego dostępu
3. **Sieci** dla komunikacji
4. **DNS** : Rozwiązanie po nazwie usługi
5. **Docker Compose** upraszcza zarządzanie

## 🔗 Następny moduł

Przejdź do modułu [7. Dobre praktyki](./07-best-practices/README.md), aby poznać najlepsze praktyki.

