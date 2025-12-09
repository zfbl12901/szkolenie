# Module 7 — Sécurité & Gouvernance

## 🎯 Objectifs
- Sécuriser un cluster PostgreSQL professionnel et gérer les permissions de manière fine et efficace.

---

## 📚 Contenu

### 1. Gestion des rôles & privilèges avancés
- Concepts de **rôles** et **utilisateurs**.
- Rôles login vs non-login.
- Attribution de privilèges :
  - `GRANT` / `REVOKE`
  - Privilèges sur tables, vues, séquences, fonctions.
- Rôles imbriqués pour simplifier la gestion des permissions.
```sql
CREATE ROLE analyst;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO analyst;
```

---

### 2. Row-Level Security (RLS)
- Contrôle d'accès sur les lignes d'une table.

#### Activation :
```sql
ALTER TABLE sales ENABLE ROW LEVEL SECURITY;
```

#### Définition de politiques :
```sql
CREATE POLICY user_sales_policy
ON sales
USING (user_id = current_user_id());
```

#### Avantages :
- Filtrage transparent pour l'utilisateur.
- Sécurité granulaire pour multi-tenancy.

---

### 3. Masquage de données
- Techniques pour protéger les données sensibles.

#### Approches possibles :
- Fonctions SQL pour anonymisation
- Vues matérialisées filtrées ou masquées

#### Exemple : masquer email sauf pour l'admin
```sql
CREATE VIEW safe_users AS
SELECT id, CASE WHEN current_role = 'admin' THEN email ELSE '***' END AS email
FROM users;
```

---

### 4. Journaux d'audit
- Suivi des actions utilisateurs pour conformité et sécurité.

#### Extension recommandée : `pg_audit`
```sql
CREATE EXTENSION IF NOT EXISTS pgaudit;
```

#### Suivi des événements :
- Connexions/déconnexions
- Modifications de données sensibles
- Accès aux tables critiques

---

### 5. Politiques de conformité
- Règles internes et réglementaires (ex : RGPD, HIPAA).

#### Bonnes pratiques :
- Séparer les rôles selon responsabilité
- Limiter les privilèges à ce qui est strictement nécessaire
- Activer RLS pour données sensibles
- Archiver et auditer les données critiques
- Mise en place de procédures automatisées pour l'audit périodique.

---

## ✅ Compétences attendues à l'issue du module
- Créer et gérer des rôles et privilèges de manière fine.
- Activer et configurer Row-Level Security pour un filtrage granulaire.
- Masquer ou anonymiser les données sensibles selon les besoins.
- Configurer et exploiter les journaux d'audit.
- Mettre en place des politiques de sécurité conformes aux réglementations.
