# 1. Prise en main Git

## 🎯 Objectifs

- Comprendre Git et le contrôle de version
- Installer Git
- Configurer Git
- Créer votre premier dépôt
- Comprendre les concepts de base

## 📋 Table des matières

1. [Introduction à Git](#introduction-à-git)
2. [Installation](#installation)
3. [Configuration](#configuration)
4. [Premier dépôt](#premier-dépôt)
5. [Concepts de base](#concepts-de-base)

---

## Introduction à Git

### Qu'est-ce que Git ?

**Git** = Système de contrôle de version distribué

- **Versioning** : Suit les modifications de fichiers
- **Distribué** : Chaque développeur a une copie complète
- **Collaboration** : Facilite le travail en équipe
- **Historique** : Conserve l'historique complet

### Pourquoi Git pour Data Analyst ?

- **Scripts** : Versionner vos scripts Python/R
- **Projets** : Gérer vos projets de portfolio
- **Collaboration** : Travailler en équipe
- **Backup** : Sauvegarder en ligne (GitHub)
- **Documentation** : Versionner votre documentation

### Git vs Autres systèmes

**Git :**
- Distribué
- Rapide
- Gratuit et open-source
- Standard de l'industrie

**Autres (SVN, CVS) :**
- Centralisé
- Plus lent
- Moins utilisé

---

## Installation

### Windows

**Étape 1 : Télécharger Git**

1. Aller sur : https://git-scm.com/download/win
2. Télécharger l'installateur
3. Exécuter l'installateur
4. Accepter les options par défaut

**Étape 2 : Vérifier l'installation**

```bash
# Ouvrir Git Bash ou PowerShell
git --version
```

### Linux

**Ubuntu/Debian :**

```bash
# Mettre à jour les paquets
sudo apt update

# Installer Git
sudo apt install git

# Vérifier
git --version
```

**CentOS/RHEL :**

```bash
# Installer Git
sudo yum install git

# Vérifier
git --version
```

### macOS

**Avec Homebrew :**

```bash
# Installer Git
brew install git

# Vérifier
git --version
```

**Ou télécharger :**

1. Aller sur : https://git-scm.com/download/mac
2. Télécharger et installer

---

## Configuration

### Configuration globale

```bash
# Configurer votre nom
git config --global user.name "Votre Nom"

# Configurer votre email
git config --global user.email "votre.email@example.com"

# Configurer l'éditeur par défaut
git config --global core.editor "code --wait"  # VS Code
# ou
git config --global core.editor "nano"  # Nano
```

### Vérifier la configuration

```bash
# Voir toute la configuration
git config --list

# Voir une configuration spécifique
git config user.name
git config user.email
```

### Configuration par dépôt

```bash
# Dans un dépôt spécifique
git config user.name "Nom pour ce projet"
git config user.email "email@example.com"
```

---

## Premier dépôt

### Créer un nouveau dépôt

```bash
# Créer un répertoire
mkdir mon-projet
cd mon-projet

# Initialiser Git
git init

# Vérifier
ls -la  # Voir le dossier .git
```

### Premier commit

```bash
# Créer un fichier
echo "# Mon Projet" > README.md

# Voir le statut
git status

# Ajouter le fichier
git add README.md

# Commiter
git commit -m "Premier commit : ajout README"
```

### Voir l'historique

```bash
# Voir les commits
git log

# Voir les commits de manière compacte
git log --oneline

# Voir les modifications
git show
```

---

## Concepts de base

### Dépôt (Repository)

**Dépôt** = Dossier avec historique Git

- **Local** : Sur votre machine
- **Distant** : Sur GitHub/GitLab
- **.git** : Dossier caché contenant l'historique

### Commit

**Commit** = Point dans l'historique

- **Snapshot** : Capture de l'état des fichiers
- **Message** : Description des modifications
- **Auteur** : Nom et email
- **Hash** : Identifiant unique (SHA-1)

### Branche (Branch)

**Branche** = Ligne de développement

- **main/master** : Branche principale
- **Autres branches** : Pour nouvelles fonctionnalités
- **Isolation** : Travail isolé

### Staging Area

**Staging Area** = Zone de préparation

- **git add** : Ajouter des fichiers
- **git commit** : Créer un commit
- **git status** : Voir l'état

---

## Exemples pratiques

### Exemple 1 : Projet Python

```bash
# Créer le projet
mkdir data-analysis
cd data-analysis
git init

# Créer les fichiers
echo "import pandas as pd" > script.py
echo "# Data Analysis Project" > README.md

# Ajouter et commiter
git add .
git commit -m "Initial commit : projet d'analyse de données"
```

### Exemple 2 : Projet avec structure

```bash
# Créer la structure
mkdir my-project
cd my-project
git init

# Créer les dossiers
mkdir src data docs

# Créer des fichiers
echo "# Mon Projet" > README.md
echo "print('Hello')" > src/main.py

# Ajouter tout
git add .

# Commiter
git commit -m "Structure initiale du projet"
```

---

## Commandes essentielles

### Informations

```bash
# Version Git
git --version

# Statut du dépôt
git status

# Historique
git log

# Configuration
git config --list
```

### Création

```bash
# Initialiser un dépôt
git init

# Cloner un dépôt
git clone <url>
```

---

## Dépannage

### Problème : Git non trouvé

**Solutions :**
1. Vérifier l'installation : `git --version`
2. Ajouter Git au PATH (Windows)
3. Réinstaller Git

### Problème : Erreur de configuration

**Solutions :**
1. Vérifier la configuration : `git config --list`
2. Reconfigurer : `git config --global user.name "Nom"`

---

## 📊 Points clés à retenir

1. **Git** = Contrôle de version distribué
2. **Dépôt** = Dossier avec historique
3. **Commit** = Point dans l'historique
4. **Branche** = Ligne de développement
5. **Staging** = Zone de préparation

## 🔗 Prochain module

Passer au module [2. Commandes de base](./02-basic-commands/README.md) pour maîtriser les commandes essentielles.

