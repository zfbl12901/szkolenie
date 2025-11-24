# 4. Dépôts distants Git

## 🎯 Objectifs

- Comprendre les dépôts distants
- Travailler avec GitHub/GitLab
- Cloner des dépôts
- Push et Pull
- Synchronisation

## 📋 Table des matières

1. [Introduction aux dépôts distants](#introduction-aux-dépôts-distants)
2. [GitHub et GitLab](#github-et-gitlab)
3. [Cloner un dépôt](#cloner-un-dépôt)
4. [Push et Pull](#push-et-pull)
5. [Synchronisation](#synchronisation)

---

## Introduction aux dépôts distants

### Qu'est-ce qu'un dépôt distant ?

**Dépôt distant** = Copie du dépôt sur un serveur

- **GitHub** : Service populaire
- **GitLab** : Alternative open-source
- **Bitbucket** : Autre option
- **Backup** : Sauvegarde en ligne

### Pourquoi un dépôt distant ?

- **Backup** : Sauvegarde automatique
- **Collaboration** : Travailler en équipe
- **Portfolio** : Présenter vos projets
- **CI/CD** : Intégration continue

---

## GitHub et GitLab

### Créer un compte GitHub

1. Aller sur : https://github.com
2. Cliquer sur "Sign up"
3. Remplir le formulaire
4. Vérifier l'email

### Créer un dépôt GitHub

1. Cliquer sur "New repository"
2. Nommer le dépôt
3. Choisir public/private
4. Ne pas initialiser avec README (si dépôt local existe)
5. Cliquer sur "Create repository"

### Créer un compte GitLab

1. Aller sur : https://gitlab.com
2. Cliquer sur "Register"
3. Remplir le formulaire
4. Vérifier l'email

---

## Cloner un dépôt

### Cloner depuis GitHub

```bash
# Cloner avec HTTPS
git clone https://github.com/username/repo.git

# Cloner avec SSH
git clone git@github.com:username/repo.git

# Cloner dans un dossier spécifique
git clone https://github.com/username/repo.git mon-dossier
```

### Cloner depuis GitLab

```bash
# Cloner avec HTTPS
git clone https://gitlab.com/username/repo.git

# Cloner avec SSH
git clone git@gitlab.com:username/repo.git
```

---

## Push et Pull

### Ajouter un remote

```bash
# Ajouter un remote
git remote add origin https://github.com/username/repo.git

# Voir les remotes
git remote -v

# Renommer un remote
git remote rename origin upstream

# Supprimer un remote
git remote remove origin
```

### Push (envoyer)

```bash
# Premier push
git push -u origin main

# Pushes suivants
git push

# Push une branche spécifique
git push origin feature-branche

# Force push (ATTENTION)
git push --force
```

### Pull (récupérer)

```bash
# Récupérer et fusionner
git pull

# Récupérer seulement
git fetch

# Fusionner après fetch
git merge origin/main
```

---

## Synchronisation

### Workflow de base

```bash
# 1. Récupérer les dernières modifications
git pull

# 2. Travailler localement
# ... modifications ...

# 3. Ajouter et commiter
git add .
git commit -m "Modifications"

# 4. Envoyer
git push
```

### Synchroniser une branche

```bash
# Créer une branche locale
git checkout -b feature-branche

# Pousser la branche
git push -u origin feature-branche

# Récupérer une branche distante
git fetch origin
git checkout -b feature-branche origin/feature-branche
```

### Mettre à jour main

```bash
# Récupérer les modifications
git fetch origin

# Fusionner
git merge origin/main

# Ou en une commande
git pull origin main
```

---

## Exemples pratiques

### Exemple 1 : Premier push

```bash
# Créer un dépôt local
mkdir mon-projet
cd mon-projet
git init

# Créer des fichiers
echo "# Mon Projet" > README.md
git add README.md
git commit -m "Initial commit"

# Ajouter le remote
git remote add origin https://github.com/username/repo.git

# Pousser
git push -u origin main
```

### Exemple 2 : Cloner et modifier

```bash
# Cloner un dépôt
git clone https://github.com/username/repo.git
cd repo

# Créer une branche
git checkout -b ma-modification

# Modifier
echo "Nouvelle ligne" >> README.md
git add README.md
git commit -m "Ajout ligne"

# Pousser
git push -u origin ma-modification
```

---

## Authentification

### HTTPS

```bash
# Première fois : demandera credentials
git push

# Stocker les credentials
git config --global credential.helper store
```

### SSH

**Générer une clé SSH :**

```bash
# Générer une clé
ssh-keygen -t ed25519 -C "votre.email@example.com"

# Copier la clé publique
cat ~/.ssh/id_ed25519.pub

# Ajouter sur GitHub/GitLab
# Settings > SSH Keys > New SSH Key
```

---

## 📊 Points clés à retenir

1. **Remote** : Dépôt sur serveur
2. **git clone** : Copier un dépôt
3. **git push** : Envoyer les modifications
4. **git pull** : Récupérer les modifications
5. **Synchronisation** : Pull avant Push

## 🔗 Prochain module

Passer au module [5. Collaboration](./05-collaboration/README.md) pour apprendre à collaborer.

