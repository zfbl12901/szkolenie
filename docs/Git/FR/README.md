# Formation Git pour Data Analyst

## 📚 Vue d'ensemble

Cette formation vous guide dans l'apprentissage de **Git** en tant que Data Analyst. Git est un système de contrôle de version distribué essentiel pour gérer le code, les scripts et la documentation de vos projets.

## 🎯 Objectifs pédagogiques

- Comprendre Git et le contrôle de version
- Installer Git
- Maîtriser les commandes de base
- Gérer les branches
- Travailler avec des dépôts distants (GitHub, GitLab)
- Collaborer sur des projets
- Utiliser Git dans vos workflows de données
- Créer des projets pratiques pour votre portfolio

## 💰 Tout est gratuit !

Cette formation utilise uniquement :
- ✅ **Git** : Open-source et gratuit
- ✅ **GitHub** : Compte gratuit (illimité)
- ✅ **GitLab** : Compte gratuit
- ✅ **Documentation officielle** : Guides complets gratuits
- ✅ **Tutoriels en ligne** : Ressources gratuites

**Budget total : 0€**

## 📖 Structure de la formation

### 1. [Prise en main Git](./01-getting-started/README.md)
   - Installer Git
   - Configuration initiale
   - Premiers dépôts
   - Concepts de base

### 2. [Commandes de base](./02-basic-commands/README.md)
   - Créer un dépôt
   - Ajouter et commiter
   - Voir l'historique
   - Annuler des modifications

### 3. [Branches](./03-branching/README.md)
   - Créer et gérer des branches
   - Fusionner des branches
   - Résoudre les conflits
   - Workflow avec branches

### 4. [Dépôts distants](./04-remote-repositories/README.md)
   - GitHub et GitLab
   - Cloner un dépôt
   - Push et Pull
   - Synchronisation

### 5. [Collaboration](./05-collaboration/README.md)
   - Fork et Pull Requests
   - Issues et Projects
   - Code Review
   - Workflow en équipe

### 6. [Fonctionnalités avancées](./06-advanced/README.md)
   - Stash
   - Rebase
   - Tags
   - Hooks

### 7. [Bonnes pratiques](./07-best-practices/README.md)
   - Messages de commit
   - Structure de projet
   - .gitignore
   - Documentation

### 8. [Projets pratiques](./08-projets/README.md)
   - Portfolio GitHub
   - Projet collaboratif
   - Gestion de scripts Python
   - Documentation de projet

## 🚀 Démarrage rapide

### Prérequis

- **Système d'exploitation** : Windows, Linux, ou macOS
- **Connexion Internet** : Pour GitHub/GitLab
- **Éditeur de texte** : VS Code, Sublime, etc.

### Installation rapide

**Windows :**
1. Télécharger Git : https://git-scm.com/download/win
2. Installer avec les options par défaut
3. Vérifier : `git --version`

**Linux :**
```bash
# Ubuntu/Debian
sudo apt install git

# CentOS/RHEL
sudo yum install git

# Vérifier
git --version
```

**macOS :**
```bash
# Avec Homebrew
brew install git

# Ou télécharger
# https://git-scm.com/download/mac

# Vérifier
git --version
```

### Configuration initiale

```bash
# Configurer votre nom
git config --global user.name "Votre Nom"

# Configurer votre email
git config --global user.email "votre.email@example.com"

# Vérifier la configuration
git config --list
```

### Premier dépôt

```bash
# Créer un nouveau dépôt
mkdir mon-projet
cd mon-projet
git init

# Créer un fichier
echo "# Mon Projet" > README.md

# Ajouter et commiter
git add README.md
git commit -m "Premier commit"
```

## 📊 Cas d'usage pour Data Analyst

- **Versioning** : Gérer les versions de vos scripts Python/R
- **Collaboration** : Travailler en équipe sur des projets
- **Portfolio** : Présenter vos projets sur GitHub
- **Documentation** : Versionner votre documentation
- **Backup** : Sauvegarder votre code en ligne

## 📚 Ressources gratuites

### Documentation officielle

- **Git Documentation** : https://git-scm.com/doc
- **GitHub Guides** : https://guides.github.com/
- **GitLab Documentation** : https://docs.gitlab.com/

### Ressources externes

- **GitHub Learning Lab** : https://lab.github.com/
- **Atlassian Git Tutorials** : https://www.atlassian.com/git/tutorials
- **YouTube** : Tutoriels Git

## 🎓 Certifications (optionnel)

### GitHub Certifications

- **GitHub Actions** : Gratuit
- **GitHub Advanced Security** : Formation gratuite

## 📝 Conventions

- Tous les exemples fonctionnent sur Windows, Linux, et macOS
- Les commandes sont identiques sur tous les systèmes
- GitHub est utilisé comme exemple principal

## 🤝 Contribution

Cette formation est conçue pour être évolutive. N'hésitez pas à proposer des améliorations.

## 📚 Ressources complémentaires

- [Git Documentation](https://git-scm.com/doc)
- [GitHub](https://github.com/)
- [GitLab](https://gitlab.com/)
- [Git Cheat Sheet](https://education.github.com/git-cheat-sheet-education.pdf)

