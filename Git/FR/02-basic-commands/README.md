# 2. Commandes de base Git

## 🎯 Objectifs

- Créer et gérer un dépôt
- Ajouter et commiter des fichiers
- Voir l'historique
- Annuler des modifications
- Ignorer des fichiers

## 📋 Table des matières

1. [Créer un dépôt](#créer-un-dépôt)
2. [Ajouter et commiter](#ajouter-et-commiter)
3. [Voir l'historique](#voir-lhistorique)
4. [Annuler des modifications](#annuler-des-modifications)
5. [.gitignore](#gitignore)

---

## Créer un dépôt

### Initialiser un dépôt local

```bash
# Créer un nouveau dépôt
mkdir mon-projet
cd mon-projet
git init

# Vérifier
ls -la  # Voir .git
```

### Cloner un dépôt existant

```bash
# Cloner depuis GitHub
git clone https://github.com/username/repo.git

# Cloner dans un dossier spécifique
git clone https://github.com/username/repo.git mon-dossier

# Cloner avec SSH
git clone git@github.com:username/repo.git
```

---

## Ajouter et commiter

### Workflow de base

```bash
# 1. Voir le statut
git status

# 2. Ajouter des fichiers
git add fichier.py
git add .  # Tous les fichiers

# 3. Commiter
git commit -m "Message de commit"
```

### Ajouter des fichiers

```bash
# Ajouter un fichier spécifique
git add script.py

# Ajouter tous les fichiers
git add .

# Ajouter tous les fichiers Python
git add *.py

# Ajouter un dossier
git add src/
```

### Créer un commit

```bash
# Commit avec message
git commit -m "Ajout du script d'analyse"

# Commit avec message détaillé
git commit -m "Titre" -m "Description détaillée"

# Commit en ajoutant automatiquement les fichiers modifiés
git commit -am "Message"
```

### Messages de commit

**Bon format :**
```
Type : Description courte (50 caractères max)

Description détaillée si nécessaire
```

**Exemples :**
```
feat: Ajout fonction analyse de données
fix: Correction bug dans le calcul
docs: Mise à jour README
refactor: Réorganisation du code
```

---

## Voir l'historique

### git log

```bash
# Historique complet
git log

# Historique compact
git log --oneline

# Historique avec graphique
git log --graph --oneline --all

# Limiter le nombre
git log -5  # 5 derniers commits

# Par auteur
git log --author="Nom"

# Par date
git log --since="2024-01-01"
```

### Voir les modifications

```bash
# Dernier commit
git show

# Commit spécifique
git show <hash>

# Différence entre commits
git diff HEAD~1 HEAD

# Différence avec staging
git diff --staged
```

### Statut

```bash
# Statut détaillé
git status

# Statut court
git status -s

# Voir les fichiers modifiés
git status --short
```

---

## Annuler des modifications

### Annuler dans le working directory

```bash
# Annuler modifications d'un fichier
git checkout -- fichier.py

# Annuler toutes les modifications
git checkout -- .

# Nouvelle syntaxe (Git 2.23+)
git restore fichier.py
```

### Annuler dans staging

```bash
# Retirer un fichier du staging
git reset HEAD fichier.py

# Retirer tous les fichiers
git reset HEAD

# Nouvelle syntaxe
git restore --staged fichier.py
```

### Modifier le dernier commit

```bash
# Modifier le message
git commit --amend -m "Nouveau message"

# Ajouter des fichiers oubliés
git add fichier_oublie.py
git commit --amend --no-edit
```

### Annuler un commit (garder les modifications)

```bash
# Annuler le dernier commit
git reset --soft HEAD~1

# Annuler et supprimer les modifications
git reset --hard HEAD~1  # ATTENTION : destructif
```

---

## .gitignore

### Créer un .gitignore

**Fichier `.gitignore` :**

```
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
venv/
env/

# Jupyter
.ipynb_checkpoints
*.ipynb

# Data
*.csv
*.xlsx
*.parquet
data/

# IDE
.vscode/
.idea/
*.swp

# OS
.DS_Store
Thumbs.db

# Secrets
.env
*.key
config.ini
```

### Utiliser .gitignore

```bash
# Créer le fichier
touch .gitignore

# Ajouter au dépôt
git add .gitignore
git commit -m "Ajout .gitignore"
```

### Patterns

```
# Ignorer un fichier
fichier.txt

# Ignorer un dossier
dossier/

# Ignorer tous les fichiers .log
*.log

# Ignorer sauf un fichier spécifique
*.log
!important.log

# Ignorer dans un dossier spécifique
data/*.csv
```

---

## Exemples pratiques

### Exemple 1 : Projet Python

```bash
# Créer le projet
mkdir data-project
cd data-project
git init

# Créer .gitignore
echo "__pycache__/" > .gitignore
echo "*.pyc" >> .gitignore
echo "venv/" >> .gitignore

# Créer des fichiers
echo "import pandas as pd" > main.py
echo "# Data Project" > README.md

# Ajouter et commiter
git add .
git commit -m "Initial commit : projet Python"
```

### Exemple 2 : Gestion des modifications

```bash
# Modifier un fichier
echo "print('Hello')" >> script.py

# Voir les modifications
git diff

# Ajouter au staging
git add script.py

# Voir les modifications staged
git diff --staged

# Commiter
git commit -m "Ajout fonction hello"
```

---

## 📊 Points clés à retenir

1. **git add** : Ajouter au staging
2. **git commit** : Créer un commit
3. **git log** : Voir l'historique
4. **git status** : Voir l'état
5. **.gitignore** : Exclure des fichiers

## 🔗 Prochain module

Passer au module [3. Branches](./03-branching/README.md) pour apprendre à gérer les branches.

