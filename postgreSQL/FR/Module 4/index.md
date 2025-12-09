# Module 4 — Fonctions, Triggers & Extensions

## 🎯 Objectifs
- Automatiser, enrichir et étendre PostgreSQL grâce aux fonctions, triggers et extensions.

---

## 📚 Contenu

### 1. PL/pgSQL avancé
- Création de fonctions stockées avec `CREATE FUNCTION`.
- Syntaxe, variables, boucles, conditions.
- Gestion des exceptions avec `EXCEPTION`.
- Optimisations :
  - Minimiser les accès disque
  - Utiliser `PERFORM` pour les appels sans retour
  - Déterminer les fonctions `IMMUTABLE`, `STABLE` ou `VOLATILE` pour le planner

```sql
CREATE OR REPLACE FUNCTION update_balance(user_id int, amount numeric)
RETURNS void AS $$
BEGIN
    UPDATE accounts
    SET balance = balance + amount
    WHERE id = user_id;
EXCEPTION WHEN OTHERS THEN
    RAISE NOTICE 'Erreur lors de la mise à jour';
END;
$$ LANGUAGE plpgsql;
```

---

### 2. Curseurs
- Pour parcourir un jeu de résultats ligne par ligne.

#### Syntaxe :
```sql
DECLARE cur_name CURSOR FOR SELECT * FROM orders;
FETCH NEXT FROM cur_name;
CLOSE cur_name;
```

- Utilisation pour traiter de gros volumes sans surcharger la mémoire.

---

### 3. Triggers
- Automatisation des actions sur tables.

#### Types :
- **BEFORE** : avant INSERT/UPDATE/DELETE
- **AFTER** : après modification
- **INSTEAD OF** : sur les vues

#### Exemple simple :
```sql
CREATE TRIGGER update_timestamp
BEFORE UPDATE ON users
FOR EACH ROW
EXECUTE FUNCTION set_updated_at();
```

---

### 4. Extensions indispensables

#### 4.1 `pg_stat_statements`
- Suivi des requêtes les plus coûteuses.
```sql
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;
SELECT * FROM pg_stat_statements ORDER BY total_time DESC LIMIT 10;
```

#### 4.2 `pg_trgm`
- Recherche floue et similarité textuelle.
```sql
CREATE EXTENSION IF NOT EXISTS pg_trgm;
SELECT * FROM users WHERE username % 'alex';
```

#### 4.3 `hstore`
- Stockage clé-valeur simple.
```sql
CREATE EXTENSION IF NOT EXISTS hstore;
```

#### 4.4 `uuid-ossp`
- Génération d'UUID pour clés primaires.
```sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
```

#### 4.5 PostGIS
- Données géospatiales : `geometry`, `geography`.
```sql
CREATE EXTENSION IF NOT EXISTS postgis;
```

#### 4.6 TimescaleDB (time-series)
- Optimisation des données temporelles.
```sql
CREATE EXTENSION IF NOT EXISTS timescaledb;
```

---

## ✅ Compétences attendues à l'issue du module
- Créer et optimiser des fonctions PL/pgSQL.
- Utiliser curseurs et gérer les exceptions efficacement.
- Automatiser des actions via triggers.
- Installer et exploiter des extensions essentielles pour l'analytics, les données géospatiales et les séries temporelles.
