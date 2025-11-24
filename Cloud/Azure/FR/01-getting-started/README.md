# 1. Prise en main Azure

## 🎯 Objectifs

- Créer un compte Azure gratuit
- Comprendre les crédits gratuits Azure
- Naviguer dans le portail Azure
- Configurer la sécurité de base (Azure AD)
- Surveiller les coûts

## 📋 Table des matières

1. [Créer un compte Azure gratuit](#créer-un-compte-azure-gratuit)
2. [Comprendre les crédits gratuits](#comprendre-les-crédits-gratuits)
3. [Naviguer dans le portail Azure](#naviguer-dans-le-portail-azure)
4. [Configuration Azure AD (sécurité)](#configuration-azure-ad-sécurité)
5. [Surveillance des coûts](#surveillance-des-coûts)

---

## Créer un compte Azure gratuit

### Étape 1 : Inscription

1. **Aller sur le site Azure**
   - URL : https://azure.microsoft.com/fr-fr/free/
   - Cliquer sur "Démarrer gratuitement"

2. **Se connecter avec Microsoft**
   - Utiliser un compte Microsoft existant
   - Ou créer un nouveau compte Microsoft

3. **Vérification d'identité**
   - Code reçu par SMS ou email
   - Entrer le code de vérification

4. **Informations personnelles**
   - Nom
   - Prénom
   - Numéro de téléphone
   - Pays

5. **Vérification par téléphone**
   - Appel automatique ou SMS
   - Entrer le code de vérification

6. **Méthode de paiement**
   - **Important** : Carte bancaire requise mais **non débitée**
   - Azure vous donne 200$ de crédit pour 30 jours
   - Après 30 jours : services gratuits permanents
   - Vous pouvez supprimer la carte après (non recommandé)

7. **Vérification d'identité finale**
   - Vérification par SMS ou appel
   - Confirmation du compte

### Étape 2 : Confirmation

- Email de confirmation reçu
- Compte Azure actif immédiatement
- Accès au portail Azure
- **200$ de crédit** disponibles pour 30 jours

**⚠️ Important :** Ne pas créer plusieurs comptes avec la même carte bancaire (risque de suspension).

---

## Comprendre les crédits gratuits

### Offre Azure gratuit

Azure offre **3 types** de services gratuits :

#### 1. Crédit de 200$ (30 jours)

**Ce que vous pouvez faire :**
- Tester n'importe quel service Azure
- Créer des machines virtuelles
- Utiliser des services payants
- Expérimenter librement

**Conditions :**
- Valable 30 jours après inscription
- Si crédit épuisé avant 30 jours : services arrêtés
- Après 30 jours : passage aux services gratuits permanents

#### 2. Services gratuits pendant 12 mois

**Services utiles pour Data Analyst :**

- **Azure SQL Database** : Gratuit jusqu'à 32 Go (12 mois)
- **Azure Storage** : 5 Go (12 mois)
- **Azure App Service** : 60 minutes/jour (12 mois)
- **Azure Functions** : 1 million d'exécutions/mois (toujours gratuit)

**Conditions :**
- Gratuit pendant 12 mois après inscription
- Limites par mois
- Au-delà : facturation normale

#### 3. Services toujours gratuits

**Services utiles pour Data Analyst :**

- **Azure Functions** : 1 million d'exécutions/mois (toujours gratuit)
- **Azure Cosmos DB** : 400 RU/s (toujours gratuit)
- **Azure Active Directory** : 50 000 objets (toujours gratuit)
- **Azure DevOps** : 5 utilisateurs (toujours gratuit)

**Conditions :**
- Gratuit indéfiniment
- Limites par mois
- Au-delà : facturation au-delà de la limite

### Vérifier vos crédits

1. Aller dans le portail Azure
2. "Cost Management + Billing"
3. Voir les crédits restants
4. Voir l'utilisation par service

---

## Naviguer dans le portail Azure

### Interface principale

**Éléments clés :**

1. **Barre de recherche** (en haut)
   - Rechercher des services rapidement
   - Exemple : taper "SQL" pour trouver SQL Database

2. **Menu Azure** (icône ☰ en haut à gauche)
   - Tous les services Azure
   - Organisés par catégorie
   - Favoris personnalisables

3. **Notifications** (en haut à droite)
   - Alertes et notifications
   - Statut des déploiements

4. **Paramètres** (en haut à droite)
   - Paramètres du compte
   - Thème (clair/sombre)
   - Langue

5. **Cloud Shell** (icône >_ en haut)
   - Terminal dans le navigateur
   - PowerShell ou Bash
   - Très utile pour les commandes

### Services essentiels pour Data Analyst

**Dans le menu Azure, chercher :**

- **Storage accounts** : Stockage de données
- **Data Factory** : ETL cloud
- **SQL databases** : Bases de données SQL
- **Synapse Analytics** : Data warehouse
- **Databricks** : Big Data analytics
- **Functions** : Serverless computing

### Première connexion

1. Se connecter : https://portal.azure.com/
2. Explorer le tableau de bord
3. Cliquer sur "Tous les services" pour voir tous les services
4. Utiliser la barre de recherche pour trouver un service
5. Épingler les services fréquents au tableau de bord

---

## Configuration Azure AD (sécurité)

### Qu'est-ce qu'Azure AD ?

**Azure AD** (Azure Active Directory) = Gestion des identités et accès

- Gérer les utilisateurs
- Gérer les permissions
- Sécuriser l'accès aux services
- Authentification multi-facteurs (MFA)

### Bonnes pratiques de sécurité

#### 1. Activer l'authentification multi-facteurs (MFA)

**Pour le compte administrateur :**

1. Aller dans Azure AD
2. "Utilisateurs" → Sélectionner votre compte
3. "Authentification multifacteur"
4. Cliquer sur "Activer"
5. Suivre les instructions

**⚠️ Important :** Toujours activer MFA pour les comptes administrateurs.

#### 2. Créer des utilisateurs Azure AD (recommandé)

**Pour le travail en équipe :**

1. Aller dans Azure AD
2. "Utilisateurs" → "Nouvel utilisateur"
3. Nom d'utilisateur : `data-analyst@votredomaine.onmicrosoft.com`
4. Mot de passe temporaire
5. Rôles : "Utilisateur" (par défaut)
6. Créer l'utilisateur

#### 3. Rôles Azure (RBAC)

**Rôles utiles pour Data Analyst :**

- **Contributeur** : Peut créer et gérer des ressources
- **Lecteur** : Peut seulement lire
- **Contributeur de compte de stockage** : Accès aux comptes de stockage
- **Contributeur SQL DB** : Accès aux bases SQL

**Attribuer un rôle :**

1. Aller à la ressource (ex: Storage Account)
2. "Contrôle d'accès (IAM)"
3. "Ajouter" → "Ajouter une attribution de rôle"
4. Sélectionner le rôle
5. Sélectionner l'utilisateur

### Politiques de sécurité recommandées

1. **Mots de passe forts**
   - Minimum 12 caractères
   - Complexité requise

2. **Expiration des mots de passe**
   - 90 jours (recommandé)

3. **Blocage de compte**
   - Après 5 tentatives échouées

---

## Surveillance des coûts

### Activer les alertes de coût

**Étape 1 : Configurer les alertes**

1. Aller dans "Cost Management + Billing"
2. "Alertes de coût"
3. "Nouvelle alerte de coût"
4. Seuil : 5€ (recommandé)
5. Email de notification

**Résultat :** Email reçu si les coûts dépassent 5€.

### Vérifier l'utilisation des crédits

1. "Cost Management + Billing"
2. "Crédits Azure"
3. Voir les crédits restants
4. Voir l'utilisation par service
5. Voir la date d'expiration (30 jours)

### Azure Cost Management

1. "Cost Management + Billing" → "Cost Management"
2. Voir les coûts par service
3. Filtrer par période
4. Exporter les rapports
5. Créer des budgets

**⚠️ Important :** Vérifier régulièrement (hebdomadaire recommandé).

### Conseils pour rester gratuit

1. **Supprimer les ressources inutilisées**
   - Arrêter les machines virtuelles non utilisées
   - Supprimer les comptes de stockage vides
   - Nettoyer les groupes de ressources

2. **Utiliser les services gratuits**
   - Privilégier les services toujours gratuits
   - Utiliser les crédits intelligemment
   - Arrêter les services non utilisés

3. **Créer des budgets**
   - "Cost Management" → "Budgets"
   - Créer un budget de 5€
   - Alertes automatiques

4. **Arrêter les services non utilisés**
   - Machines virtuelles : arrêter quand non utilisées
   - Bases de données : arrêter ou mettre en pause
   - Comptes de stockage : supprimer si vides

### Groupes de ressources

**Organiser vos ressources :**

1. Créer un groupe de ressources : `rg-data-analyst-training`
2. Toutes les ressources de formation dans ce groupe
3. Facilite la suppression en une fois
4. Facilite la gestion des coûts

**Créer un groupe de ressources :**

1. "Groupes de ressources" → "Ajouter"
2. Nom : `rg-data-analyst-training`
3. Région : Choisir la région la plus proche
4. Créer

---

## 📊 Points clés à retenir

1. **Compte Azure gratuit** : 200$ de crédit (30 jours) + services gratuits
2. **Crédits gratuits** : 3 types (200$, 12 mois, toujours gratuit)
3. **Sécurité Azure AD** : Activer MFA, créer utilisateurs
4. **Surveillance** : Alertes de coût essentielles
5. **Rester gratuit** : Supprimer ressources inutilisées, utiliser groupes de ressources

## 🔗 Prochain module

Passer au module [2. Azure Storage - Stockage de données](../02-storage/README.md) pour apprendre à stocker des données sur Azure.

