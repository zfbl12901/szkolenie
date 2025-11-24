# 3. Branches Git

## 🎯 Objectifs

- Comprendre les branches
- Créer et gérer des branches
- Fusionner des branches
- Résoudre les conflits
- Workflow avec branches

## 📋 Table des matières

1. [Introduction aux branches](#introduction-aux-branches)
2. [Créer des branches](#créer-des-branches)
3. [Fusionner des branches](#fusionner-des-branches)
4. [Résoudre les conflits](#résoudre-les-conflits)
5. [Workflow](#workflow)

---

## Introduction aux branches

### Qu'est-ce qu'une branche ?

**Branche** = Ligne de développement indépendante

- **Isolation** : Travail isolé
- **Parallèle** : Plusieurs branches en même temps
- **Fusion** : Combiner les modifications
- **main/master** : Branche principale

### Pourquoi utiliser des branches ?

- **Nouvelles fonctionnalités** : Une branche par fonctionnalité
- **Corrections** : Branche pour les bugs
- **Expérimentation** : Tester sans risque
- **Collaboration** : Travail en parallèle

---

## Créer des branches

### Créer une nouvelle branche

```bash
# Créer une branche
git branch feature-analyse

# Créer et basculer
git checkout -b feature-analyse

# Nouvelle syntaxe (Git 2.23+)
git switch -c feature-analyse
```

### Basculer entre branches

```bash
# Basculer vers une branche
git checkout feature-analyse

# Nouvelle syntaxe
git switch feature-analyse

# Retourner à main
git checkout main
git switch main
```

### Lister les branches

```bash
# Branches locales
git branch

# Branches distantes
git branch -r

# Toutes les branches
git branch -a

# Avec dernier commit
git branch -v
```

### Supprimer une branche

```bash
# Supprimer une branche locale
git branch -d feature-analyse

# Forcer la suppression
git branch -D feature-analyse

# Supprimer une branche distante
git push origin --delete feature-analyse
```

---

## Fusionner des branches

### Merge (fusion)

```bash
# Basculer sur main
git checkout main

# Fusionner la branche
git merge feature-analyse

# Fusionner avec message
git merge feature-analyse -m "Fusion feature analyse"
```

### Types de merge

**Fast-forward :**
- Pas de conflit
- Historique linéaire
- Simple

**Merge commit :**
- Crée un commit de fusion
- Conserve l'historique
- Plus complexe

### Rebase (alternative)

```bash
# Rebase interactif
git rebase main

# Rebase interactif avec édition
git rebase -i HEAD~3
```

---

## Résoudre les conflits

### Quand surviennent les conflits ?

- **Même ligne modifiée** : Dans deux branches différentes
- **Fichier supprimé** : Dans une branche, modifié dans l'autre
- **Fichier ajouté** : Même nom dans deux branches

### Résoudre un conflit

**Étape 1 : Identifier le conflit**

```bash
# Voir les fichiers en conflit
git status
```

**Étape 2 : Ouvrir le fichier**

```python
# Fichier avec conflit
<<<<<<< HEAD
print("Version main")
=======
print("Version feature")
>>>>>>> feature-analyse
```

**Étape 3 : Résoudre manuellement**

```python
# Choisir la version ou combiner
print("Version combinée")
```

**Étape 4 : Marquer comme résolu**

```bash
# Ajouter le fichier résolu
git add fichier.py

# Finaliser le merge
git commit
```

### Outils de résolution

```bash
# Ouvrir un outil de merge
git mergetool

# Voir les conflits
git diff
```

---

## Workflow

### Workflow simple

```bash
# 1. Créer une branche pour une fonctionnalité
git checkout -b feature-nouvelle-fonction

# 2. Travailler sur la branche
# ... modifications ...

# 3. Commiter
git add .
git commit -m "Ajout nouvelle fonctionnalité"

# 4. Fusionner dans main
git checkout main
git merge feature-nouvelle-fonction

# 5. Supprimer la branche
git branch -d feature-nouvelle-fonction
```

### Git Flow (avancé)

```bash
# Branches principales
main        # Production
develop     # Développement

# Branches de support
feature/*   # Nouvelles fonctionnalités
hotfix/*    # Corrections urgentes
release/*   # Préparation release
```

---

## Exemples pratiques

### Exemple 1 : Nouvelle fonctionnalité

```bash
# Créer branche
git checkout -b feature-analyse-donnees

# Travailler
echo "def analyse():" > analyse.py
git add analyse.py
git commit -m "Ajout fonction analyse"

# Fusionner
git checkout main
git merge feature-analyse-donnees
```

### Exemple 2 : Correction de bug

```bash
# Créer branche hotfix
git checkout -b hotfix-bug-calcul

# Corriger
# ... modifications ...

# Commiter
git add .
git commit -m "Correction bug calcul"

# Fusionner rapidement
git checkout main
git merge hotfix-bug-calcul
```

---

## 📊 Points clés à retenir

1. **Branches** : Lignes de développement isolées
2. **git branch** : Créer/gérer branches
3. **git merge** : Fusionner branches
4. **Conflits** : Résoudre manuellement
5. **Workflow** : Une branche par fonctionnalité

## 🔗 Prochain module

Passer au module [4. Dépôts distants](./04-remote-repositories/README.md) pour travailler avec GitHub/GitLab.

