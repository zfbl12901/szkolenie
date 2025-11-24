# 7. Najlepsze praktyki

## 🎯 Cele

- Modelować dane efektywnie
- Wybierać strategie partycjonowania
- Zarządzać pamięcią
- Zabezpieczać dostęp

## Modelowanie danych

### Wybierać odpowiednie typy

```sql
-- ✅ Dobrze : UInt32 dla ID
user_id UInt32

-- ❌ Mniej dobrze : UInt64 niepotrzebne
user_id UInt64

-- ✅ Dobrze : Date dla dat
event_date Date

-- ❌ Mniej dobrze : String dla dat
event_date String
```

## Strategie partycjonowania

### Według daty (zalecane)

```sql
PARTITION BY toYYYYMM(event_date)
```

### Unikać zbyt wielu partycji

```sql
-- ✅ Dobrze : Partycja miesięczna
PARTITION BY toYYYYMM(date)

-- ❌ Mniej dobrze : Partycja dzienna (zbyt wiele)
PARTITION BY date
```

## Zarządzanie pamięcią

### LIMIT zapytań

```sql
-- ✅ Dobrze
SELECT * FROM events LIMIT 1000;

-- ❌ Mniej dobrze
SELECT * FROM events;
```

## Bezpieczeństwo

### Tworzenie użytkowników

```sql
CREATE USER analyst IDENTIFIED BY 'password';
GRANT SELECT ON analytics.* TO analyst;
```

### Szczegółowe uprawnienia

```sql
GRANT SELECT ON analytics.events TO analyst;
GRANT INSERT ON analytics.temp_table TO analyst;
```

## Konserwacja

### Sprawdzać partycje

```sql
SELECT 
    partition,
    rows,
    formatReadableSize(bytes_on_disk) as size
FROM system.parts
WHERE active
ORDER BY partition;
```

### Czyścić stare dane

```sql
ALTER TABLE events DROP PARTITION '202301';
```

### Optymalizować tabele

```sql
OPTIMIZE TABLE events FINAL;
```

---

**Następny krok :** [Projekty praktyczne](./08-projets/README.md)

