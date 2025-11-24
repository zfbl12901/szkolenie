# 6. Fonctionnalités avancées Git

## 🎯 Objectifs

- Utiliser Stash
- Comprendre Rebase
- Gérer les Tags
- Utiliser les Hooks
- Commandes avancées

## 📋 Table des matières

1. [Stash](#stash)
2. [Rebase](#rebase)
3. [Tags](#tags)
4. [Hooks](#hooks)
5. [Commandes avancées](#commandes-avancées)

---

## Stash

### Qu'est-ce que Stash ?

**Stash** = Sauvegarder temporairement des modifications

- **Temporaire** : Modifications non commitées
- **Rapide** : Basculer de branche rapidement
- **Récupérable** : Récupérer plus tard

### Utiliser Stash

```bash
# Sauvegarder les modifications
git stash

# Avec message
git stash save "Message descriptif"

# Inclure les fichiers non trackés
git stash -u

# Voir la liste
git stash list

# Appliquer le dernier stash
git stash apply

# Appliquer et supprimer
git stash pop

# Appliquer un stash spécifique
git stash apply stash@{0}

# Supprimer un stash
git stash drop stash@{0}

# Supprimer tous les stashes
git stash clear
```

### Exemple

```bash
# Travailler sur une branche
git checkout feature-analyse
# ... modifications non commitées ...

# Besoin de basculer rapidement
git stash

# Basculer sur main
git checkout main
# ... faire quelque chose ...

# Retourner et récupérer
git checkout feature-analyse
git stash pop
```

---

## Rebase

### Qu'est-ce que Rebase ?

**Rebase** = Réappliquer les commits sur une autre base

- **Historique linéaire** : Plus propre
- **Réécriture** : Modifie l'historique
- **Attention** : Ne pas rebase sur branches partagées

### Rebase simple

```bash
# Rebase sur main
git checkout feature-branche
git rebase main

# Rebase interactif
git rebase -i HEAD~3
```

### Rebase interactif

```bash
# Éditer les 3 derniers commits
git rebase -i HEAD~3

# Options :
# pick : Garder le commit
# reword : Modifier le message
# edit : Modifier le commit
# squash : Fusionner avec le précédent
# drop : Supprimer le commit
```

### Résoudre les conflits pendant rebase

```bash
# Si conflit pendant rebase
# 1. Résoudre le conflit
# 2. Ajouter le fichier
git add fichier.py

# 3. Continuer le rebase
git rebase --continue

# Ou annuler
git rebase --abort
```

---

## Tags

### Qu'est-ce qu'un Tag ?

**Tag** = Pointeur vers un commit spécifique

- **Version** : Marquer des versions
- **Release** : Points de release
- **Référence** : Référence stable

### Créer un Tag

```bash
# Tag léger
git tag v1.0.0

# Tag annoté (recommandé)
git tag -a v1.0.0 -m "Version 1.0.0"

# Tag sur un commit spécifique
git tag -a v1.0.0 <hash> -m "Message"

# Voir les tags
git tag

# Voir les détails
git show v1.0.0
```

### Pousser les Tags

```bash
# Pousser un tag
git push origin v1.0.0

# Pousser tous les tags
git push origin --tags
```

### Supprimer un Tag

```bash
# Supprimer localement
git tag -d v1.0.0

# Supprimer sur le remote
git push origin --delete v1.0.0
```

---

## Hooks

### Qu'est-ce qu'un Hook ?

**Hook** = Script exécuté à certains événements

- **Automatisation** : Exécuter des actions
- **Validation** : Vérifier avant commit
- **Notification** : Notifier après push

### Hooks disponibles

**Pre-commit :**
- Avant le commit
- Validation du code
- Tests

**Post-commit :**
- Après le commit
- Notifications

**Pre-push :**
- Avant le push
- Tests complets

### Exemple de Hook

**`.git/hooks/pre-commit` :**

```bash
#!/bin/bash
# Exécuter les tests avant commit
python -m pytest tests/

# Si échec, annuler le commit
if [ $? -ne 0 ]; then
    echo "Tests échoués, commit annulé"
    exit 1
fi
```

---

## Commandes avancées

### Cherry-pick

```bash
# Appliquer un commit spécifique
git cherry-pick <hash>

# Appliquer plusieurs commits
git cherry-pick <hash1> <hash2>
```

### Reflog

```bash
# Voir l'historique des actions
git reflog

# Récupérer un commit perdu
git checkout <hash>
```

### Bisect

```bash
# Trouver le commit qui a introduit un bug
git bisect start
git bisect bad  # Commit actuel est mauvais
git bisect good <hash>  # Commit connu bon

# Git va tester des commits
# Marquer comme good ou bad
git bisect good
git bisect bad

# Terminer
git bisect reset
```

### Submodules

```bash
# Ajouter un submodule
git submodule add https://github.com/user/repo.git path

# Initialiser les submodules
git submodule init
git submodule update

# En une commande
git submodule update --init --recursive
```

---

## Exemples pratiques

### Exemple 1 : Stash pour changement urgent

```bash
# Travailler sur feature
git checkout feature-analyse
# ... modifications ...

# Bug urgent à corriger
git stash
git checkout main
git checkout -b hotfix-bug

# Corriger
# ... modifications ...
git commit -m "fix: Bug urgent"
git checkout main
git merge hotfix-bug

# Retourner au travail
git checkout feature-analyse
git stash pop
```

### Exemple 2 : Tag pour release

```bash
# Préparer la release
git checkout main
git pull origin main

# Créer le tag
git tag -a v1.0.0 -m "Release version 1.0.0"

# Pousser
git push origin main
git push origin v1.0.0
```

---

## 📊 Points clés à retenir

1. **Stash** : Sauvegarder temporairement
2. **Rebase** : Réécrire l'historique
3. **Tags** : Marquer des versions
4. **Hooks** : Automatiser des actions
5. **Commandes avancées** : Outils puissants

## 🔗 Prochain module

Passer au module [7. Bonnes pratiques](./07-best-practices/README.md) pour les meilleures pratiques.

