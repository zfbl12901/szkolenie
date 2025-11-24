# 5. Collaboration avec Git

## 🎯 Objectifs

- Fork et Pull Requests
- Issues et Projects
- Code Review
- Workflow en équipe
- Bonnes pratiques

## 📋 Table des matières

1. [Fork](#fork)
2. [Pull Requests](#pull-requests)
3. [Issues](#issues)
4. [Code Review](#code-review)
5. [Workflow en équipe](#workflow-en-équipe)

---

## Fork

### Qu'est-ce qu'un Fork ?

**Fork** = Copie d'un dépôt dans votre compte

- **Copie complète** : Tous les fichiers et historique
- **Indépendant** : Modifications sans affecter l'original
- **Contribution** : Proposer des modifications via PR

### Forker un dépôt

**Sur GitHub :**

1. Aller sur le dépôt
2. Cliquer sur "Fork"
3. Choisir votre compte
4. Le dépôt est copié

**Cloner votre fork :**

```bash
# Cloner votre fork
git clone https://github.com/votre-username/repo.git

# Ajouter l'original comme upstream
git remote add upstream https://github.com/original-owner/repo.git

# Voir les remotes
git remote -v
```

### Synchroniser avec l'original

```bash
# Récupérer les modifications de l'original
git fetch upstream

# Fusionner dans votre branche
git checkout main
git merge upstream/main

# Pousser vers votre fork
git push origin main
```

---

## Pull Requests

### Créer une Pull Request

**Étape 1 : Créer une branche**

```bash
# Dans votre fork
git checkout -b feature-ma-contribution
```

**Étape 2 : Faire des modifications**

```bash
# Modifier les fichiers
# ... modifications ...

# Commiter
git add .
git commit -m "Ajout nouvelle fonctionnalité"
```

**Étape 3 : Pousser la branche**

```bash
# Pousser vers votre fork
git push -u origin feature-ma-contribution
```

**Étape 4 : Créer la PR sur GitHub**

1. Aller sur votre fork
2. Cliquer sur "Compare & pull request"
3. Remplir le formulaire
4. Cliquer sur "Create pull request"

### Bonnes pratiques pour PR

**Titre clair :**
```
feat: Ajout fonction analyse de données
fix: Correction bug calcul
```

**Description détaillée :**
- Ce qui a été fait
- Pourquoi
- Comment tester
- Screenshots si applicable

---

## Issues

### Créer une Issue

**Sur GitHub :**

1. Aller sur le dépôt
2. Cliquer sur "Issues"
3. Cliquer sur "New Issue"
4. Remplir le formulaire

### Types d'Issues

**Bug Report :**
- Description du bug
- Étapes pour reproduire
- Comportement attendu
- Environnement

**Feature Request :**
- Description de la fonctionnalité
- Cas d'usage
- Avantages

**Question :**
- Question claire
- Contexte

### Labels et Milestones

**Labels :**
- `bug` : Bug à corriger
- `enhancement` : Amélioration
- `documentation` : Documentation
- `good first issue` : Pour débutants

**Milestones :**
- Regrouper les issues
- Suivre la progression

---

## Code Review

### Processus de Review

1. **Créer la PR** : Avec description claire
2. **Attendre la review** : Les maintainers vérifient
3. **Corriger** : Si demandé
4. **Approuver** : Une fois validé
5. **Fusionner** : Par le maintainer

### Répondre aux commentaires

```bash
# Faire des modifications
# ... modifications ...

# Commiter
git add .
git commit -m "Correction selon review"

# Pousser
git push
```

### Bonnes pratiques

- **Code clair** : Lisible et commenté
- **Tests** : Ajouter des tests
- **Documentation** : Mettre à jour la doc
- **Respecter le style** : Suivre les conventions

---

## Workflow en équipe

### Workflow standard

```bash
# 1. Récupérer les dernières modifications
git pull origin main

# 2. Créer une branche
git checkout -b feature-nouvelle-fonction

# 3. Travailler
# ... modifications ...

# 4. Commiter régulièrement
git add .
git commit -m "Message clair"

# 5. Pousser la branche
git push -u origin feature-nouvelle-fonction

# 6. Créer une PR
# Sur GitHub/GitLab

# 7. Après fusion, nettoyer
git checkout main
git pull origin main
git branch -d feature-nouvelle-fonction
```

### Workflow avec plusieurs contributeurs

```bash
# Synchroniser avant de travailler
git fetch origin
git merge origin/main

# Travailler sur votre branche
git checkout -b ma-contribution

# Pousser régulièrement
git push origin ma-contribution
```

---

## Exemples pratiques

### Exemple 1 : Contribuer à un projet open-source

```bash
# 1. Forker le projet (sur GitHub)

# 2. Cloner votre fork
git clone https://github.com/votre-username/projet.git
cd projet

# 3. Ajouter l'original
git remote add upstream https://github.com/original/projet.git

# 4. Créer une branche
git checkout -b fix-bug-123

# 5. Corriger
# ... modifications ...

# 6. Commiter et pousser
git add .
git commit -m "fix: Correction bug #123"
git push -u origin fix-bug-123

# 7. Créer PR sur GitHub
```

### Exemple 2 : Travailler en équipe

```bash
# Récupérer les modifications de l'équipe
git pull origin main

# Créer votre branche
git checkout -b feature-analyse

# Travailler
# ... modifications ...

# Pousser
git push -u origin feature-analyse

# Créer PR pour review
```

---

## 📊 Points clés à retenir

1. **Fork** : Copie d'un dépôt
2. **Pull Request** : Proposer des modifications
3. **Issues** : Suivre les problèmes
4. **Code Review** : Vérification du code
5. **Workflow** : Processus structuré

## 🔗 Prochain module

Passer au module [6. Fonctionnalités avancées](./06-advanced/README.md) pour approfondir.

