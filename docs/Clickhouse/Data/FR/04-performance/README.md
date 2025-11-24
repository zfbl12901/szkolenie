# 4. Performance et Optimisation

## 🎯 Objectifs

- Comprendre les index ClickHouse
- Optimiser les requêtes
- Gérer la compression
- Monitorer les performances

## Index et projections

### Index primaire (ORDER BY)

```sql
CREATE TABLE events
(
    id UInt64,
    event_date Date,
    user_id UInt32
)
ENGINE = MergeTree()
ORDER BY (event_date, user_id);  -- Index primaire
```

### Projections (ClickHouse 21.3+)

```sql
ALTER TABLE events
ADD PROJECTION projection_by_user
(
    SELECT 
        user_id,
        event_date,
        COUNT(*) as count
    GROUP BY user_id, event_date
);
```

## Optimisation des requêtes

### Utiliser WHERE sur colonnes indexées

```sql
-- ✅ Bon : utilise l'index
SELECT * FROM events 
WHERE event_date = '2024-01-15';

-- ❌ Moins bon : scan complet
SELECT * FROM events 
WHERE value > 100;
```

### LIMIT pour limiter les résultats

```sql
SELECT * FROM events 
ORDER BY event_date DESC 
LIMIT 100;
```

### Éviter SELECT *

```sql
-- ✅ Bon
SELECT event_date, COUNT(*) 
FROM events 
GROUP BY event_date;

-- ❌ Moins bon
SELECT * FROM events;
```

## Compression

### Vérifier la compression

```sql
SELECT 
    table,
    formatReadableSize(sum(data_compressed_bytes)) as compressed,
    formatReadableSize(sum(data_uncompressed_bytes)) as uncompressed,
    round(sum(data_uncompressed_bytes) / sum(data_compressed_bytes), 2) as ratio
FROM system.parts
WHERE active
GROUP BY table;
```

### Types de compression

- **LZ4** : Rapide, compression moyenne (par défaut)
- **ZSTD** : Plus lent, meilleure compression

```sql
SETTINGS compression_codec = 'ZSTD(3)';
```

## Monitoring

### Requêtes lentes

```sql
SELECT 
    query,
    query_duration_ms,
    read_rows,
    read_bytes
FROM system.query_log
WHERE type = 'QueryFinish'
ORDER BY query_duration_ms DESC
LIMIT 10;
```

### Utilisation disque

```sql
SELECT 
    database,
    table,
    formatReadableSize(sum(bytes)) as size
FROM system.parts
WHERE active
GROUP BY database, table
ORDER BY size DESC;
```

## Bonnes pratiques

1. **Partitionner par date** pour les données temporelles
2. **ORDER BY** sur colonnes fréquemment filtrées
3. **Éviter les colonnes trop larges** (String vs FixedString)
4. **Utiliser les types appropriés** (UInt32 vs UInt64)
5. **Monitorer régulièrement** les performances

---

**Prochaine étape :** [Fonctions Avancées](./05-fonctions-avancees/README.md)

