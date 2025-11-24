# 1. Prise en main AWS

## 🎯 Objectifs

- Créer un compte AWS gratuit
- Comprendre le Free Tier AWS
- Naviguer dans la console AWS
- Configurer la sécurité de base (IAM)
- Surveiller les coûts

## 📋 Table des matières

1. [Créer un compte AWS gratuit](#créer-un-compte-aws-gratuit)
2. [Comprendre le Free Tier](#comprendre-le-free-tier)
3. [Naviguer dans la console AWS](#naviguer-dans-la-console-aws)
4. [Configuration IAM (sécurité)](#configuration-iam-sécurité)
5. [Surveillance des coûts](#surveillance-des-coûts)

---

## Créer un compte AWS gratuit

### Étape 1 : Inscription

1. **Aller sur le site AWS**
   - URL : https://aws.amazon.com/fr/free/
   - Cliquer sur "Créer un compte gratuit"

2. **Remplir le formulaire**
   - Email
   - Mot de passe (fort)
   - Nom du compte AWS

3. **Informations de contact**
   - Nom complet
   - Numéro de téléphone
   - Pays

4. **Vérification**
   - Code reçu par SMS
   - Entrer le code de vérification

5. **Méthode de paiement**
   - **Important** : Carte bancaire requise mais **non débitée**
   - AWS ne facture rien tant que vous restez dans le Free Tier
   - Vous pouvez supprimer la carte après (non recommandé)

6. **Vérification d'identité**
   - Appel automatique
   - Entrer le code à 4 chiffres

7. **Plan de support**
   - Choisir "Plan de base" (gratuit)
   - Les autres plans sont payants

### Étape 2 : Confirmation

- Email de confirmation reçu
- Compte AWS actif immédiatement
- Accès à la console AWS

**⚠️ Important :** Ne pas créer plusieurs comptes avec la même carte bancaire (risque de suspension).

---

## Comprendre le Free Tier

### Types de Free Tier

AWS offre **3 types** de services gratuits :

#### 1. Services gratuits pendant 12 mois

**Services utiles pour Data Analyst :**

- **Amazon EC2** : 750 heures/mois (t2.micro)
- **Amazon RDS** : 750 heures/mois
- **Amazon Redshift** : 750 heures/mois (2 mois seulement)
- **Amazon Elasticsearch** : 750 heures/mois

**Conditions :**
- Gratuit pendant 12 mois après inscription
- Limites par mois
- Au-delà : facturation normale

#### 2. Services toujours gratuits (avec limites)

**Services utiles pour Data Analyst :**

- **Amazon S3** : 5 Go de stockage (toujours gratuit)
- **AWS Lambda** : 1 million de requêtes/mois (toujours gratuit)
- **AWS Glue** : 10 000 objets/mois (toujours gratuit)
- **Amazon Athena** : 10 Go de données scannées/mois (toujours gratuit)
- **Amazon CloudWatch** : 10 métriques personnalisées (toujours gratuit)

**Conditions :**
- Gratuit indéfiniment
- Limites par mois
- Au-delà : facturation au-delà de la limite

#### 3. Essais gratuits à court terme

- **Amazon Redshift** : 2 mois gratuit
- **Amazon QuickSight** : 1 utilisateur gratuit

### Vérifier votre Free Tier

1. Aller dans la console AWS
2. Menu "Services" → "Billing"
3. Cliquer sur "Free Tier"
4. Voir l'utilisation par service

---

## Naviguer dans la console AWS

### Interface principale

**Éléments clés :**

1. **Barre de recherche** (en haut)
   - Rechercher des services rapidement
   - Exemple : taper "S3" pour accéder à Amazon S3

2. **Menu Services** (en haut à gauche)
   - Tous les services AWS
   - Organisés par catégorie

3. **Région** (en haut à droite)
   - Choisir la région AWS
   - **Recommandation** : Choisir la région la plus proche
   - Exemple : `eu-west-3` (Paris) pour la France

4. **Nom du compte** (en haut à droite)
   - Paramètres du compte
   - Facturation
   - Support

### Services essentiels pour Data Analyst

**Dans le menu Services, chercher :**

- **S3** : Stockage de données
- **Glue** : ETL serverless
- **Redshift** : Data warehouse
- **Athena** : Requêtes SQL sur S3
- **Lambda** : Traitement serverless
- **IAM** : Gestion des accès

### Première connexion

1. Se connecter : https://console.aws.amazon.com/
2. Explorer le tableau de bord
3. Cliquer sur "Services" pour voir tous les services
4. Utiliser la barre de recherche pour trouver un service

---

## Configuration IAM (sécurité)

### Qu'est-ce que IAM ?

**IAM** (Identity and Access Management) = Gestion des accès et identités

- Créer des utilisateurs
- Gérer les permissions
- Sécuriser l'accès aux services

### Bonnes pratiques de sécurité

#### 1. Activer l'authentification à deux facteurs (MFA)

**Pour le compte root :**

1. Aller dans IAM
2. Cliquer sur "Activate MFA"
3. Choisir un appareil (téléphone)
4. Scanner le QR code avec une app MFA
5. Entrer les codes de vérification

**⚠️ Important :** Toujours activer MFA pour le compte root.

#### 2. Créer un utilisateur IAM (recommandé)

**Ne pas utiliser le compte root pour le travail quotidien.**

1. Aller dans IAM
2. Cliquer sur "Users" → "Add users"
3. Nom d'utilisateur : `data-analyst`
4. Type d'accès : "Programmatic access" + "AWS Management Console access"
5. Permissions : "Attach existing policies directly"
   - Sélectionner : `PowerUserAccess` (pour débuter)
   - Ou créer des permissions personnalisées
6. Créer l'utilisateur
7. **Sauvegarder les identifiants** (clé d'accès + secret)

#### 3. Groupes IAM (optionnel)

Créer des groupes pour organiser les utilisateurs :

1. IAM → "Groups" → "Create group"
2. Nom : `DataAnalystGroup`
3. Attacher des politiques
4. Ajouter des utilisateurs au groupe

### Politiques IAM recommandées pour Data Analyst

**Politiques essentielles :**

- `AmazonS3FullAccess` : Accès complet à S3
- `AWSGlueServiceRole` : Accès à Glue
- `AmazonRedshiftFullAccess` : Accès à Redshift
- `AmazonAthenaFullAccess` : Accès à Athena
- `AWSLambdaFullAccess` : Accès à Lambda

**⚠️ Principe du moindre privilège :** Donner uniquement les permissions nécessaires.

---

## Surveillance des coûts

### Activer les alertes de facturation

**Étape 1 : Activer les alertes**

1. Aller dans "Billing" → "Preferences"
2. Activer "Receive Billing Alerts"
3. Activer "Receive Free Tier Usage Alerts"

**Étape 2 : Créer une alerte CloudWatch**

1. Aller dans CloudWatch
2. "Alarms" → "Create alarm"
3. Métrique : "EstimatedCharges"
4. Seuil : 5€ (recommandé)
5. Notification : Email

**Résultat :** Email reçu si les coûts dépassent 5€.

### Vérifier l'utilisation Free Tier

1. "Billing" → "Free Tier"
2. Voir l'utilisation par service
3. Vérifier les limites restantes
4. Surveiller les dates d'expiration (12 mois)

### AWS Cost Explorer

1. "Billing" → "Cost Explorer"
2. Voir les coûts par service
3. Filtrer par période
4. Exporter les rapports

**⚠️ Important :** Vérifier régulièrement (hebdomadaire recommandé).

### Conseils pour rester gratuit

1. **Supprimer les ressources inutilisées**
   - Arrêter les instances EC2 non utilisées
   - Supprimer les buckets S3 vides
   - Nettoyer les snapshots

2. **Respecter les limites Free Tier**
   - Lire attentivement les conditions
   - Surveiller l'utilisation
   - Mettre des alertes

3. **Utiliser les régions gratuites**
   - Certaines régions offrent plus de services gratuits
   - Vérifier la disponibilité

4. **Arrêter les services non utilisés**
   - Redshift : arrêter le cluster quand non utilisé
   - EC2 : arrêter les instances
   - RDS : arrêter les bases de données

---

## 📊 Points clés à retenir

1. **Compte AWS gratuit** : 200$ de crédit + Free Tier
2. **Free Tier** : 3 types (12 mois, toujours gratuit, essais)
3. **Sécurité IAM** : Activer MFA, créer utilisateurs
4. **Surveillance** : Alertes de facturation essentielles
5. **Rester gratuit** : Supprimer ressources inutilisées

## 🔗 Prochain module

Passer au module [2. Amazon S3 - Stockage de données](../02-s3/README.md) pour apprendre à stocker des données sur AWS.

