# 7. Bonnes Pratiques

## 🎯 Objectifs

- Modéliser les données efficacement
- Choisir les stratégies de partitionnement
- Gérer la mémoire
- Sécuriser l'accès

## Modélisation des données

### Choisir les bons types

```sql
-- ✅ Bon : UInt32 pour IDs
user_id UInt32

-- ❌ Moins bon : UInt64 inutile
user_id UInt64

-- ✅ Bon : Date pour dates
event_date Date

-- ❌ Moins bon : String pour dates
event_date String
```

### Éviter les colonnes trop larges

```sql
-- ✅ Bon : String pour textes variables
description String

-- ❌ Moins bon : FixedString trop large
description FixedString(10000)
```

## Stratégies de partitionnement

### Par date (recommandé)

```sql
PARTITION BY toYYYYMM(event_date)
```

### Par hash pour distribution

```sql
PARTITION BY intHash32(user_id) % 10
```

### Éviter trop de partitions

```sql
-- ✅ Bon : Partition mensuelle
PARTITION BY toYYYYMM(date)

-- ❌ Moins bon : Partition quotidienne (trop de partitions)
PARTITION BY date
```

## Gestion de la mémoire

### LIMIT les requêtes

```sql
-- ✅ Bon
SELECT * FROM events LIMIT 1000;

-- ❌ Moins bon
SELECT * FROM events;
```

### Éviter les SELECT *

```sql
-- ✅ Bon
SELECT event_date, COUNT(*) FROM events;

-- ❌ Moins bon
SELECT * FROM events;
```

## Sécurité

### Créer des utilisateurs

```sql
CREATE USER analyst IDENTIFIED BY 'password';
GRANT SELECT ON analytics.* TO analyst;
```

### Permissions granulaires

```sql
GRANT SELECT ON analytics.events TO analyst;
GRANT INSERT ON analytics.temp_table TO analyst;
```

## Maintenance

### Vérifier les partitions

```sql
SELECT 
    partition,
    rows,
    formatReadableSize(bytes_on_disk) as size
FROM system.parts
WHERE active
ORDER BY partition;
```

### Nettoyer les anciennes données

```sql
ALTER TABLE events DROP PARTITION '202301';
```

### Optimiser les tables

```sql
OPTIMIZE TABLE events FINAL;
```

## Checklist

- [ ] Types de données appropriés
- [ ] Partitionnement configuré
- [ ] ORDER BY optimisé
- [ ] Index sur colonnes filtrées
- [ ] Requêtes avec LIMIT
- [ ] Utilisateurs et permissions
- [ ] Monitoring en place

---

**Prochaine étape :** [Projets Pratiques](./08-projets/README.md)

