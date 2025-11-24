# 8. Projets pratiques Git

## 🎯 Objectifs

- Créer un portfolio GitHub
- Gérer un projet collaboratif
- Versionner des scripts Python
- Documenter un projet
- Projets pour portfolio

## 📋 Table des matières

1. [Projet 1 : Portfolio GitHub](#projet-1--portfolio-github)
2. [Projet 2 : Projet collaboratif](#projet-2--projet-collaboratif)
3. [Projet 3 : Scripts Python versionnés](#projet-3--scripts-python-versionnés)
4. [Projet 4 : Documentation de projet](#projet-4--documentation-de-projet)

---

## Projet 1 : Portfolio GitHub

### Objectif

Créer un portfolio professionnel sur GitHub.

### Structure

```
portfolio/
├── README.md
├── projects/
│   ├── project1/
│   ├── project2/
│   └── project3/
├── scripts/
│   └── utilities.py
└── docs/
    └── resume.md
```

### README.md

```markdown
# Mon Portfolio Data Analyst

## À propos

Data Analyst passionné par l'analyse de données et la visualisation.

## Projets

### [Projet 1 : Analyse de ventes](projects/project1/)
Analyse des ventes avec Python et pandas.

### [Projet 2 : Dashboard PowerBI](projects/project2/)
Dashboard interactif pour la gestion.

## Compétences

- Python
- SQL
- PowerBI
- Git/GitHub

## Contact

Email: votre.email@example.com
LinkedIn: linkedin.com/in/votre-profil
```

### Créer le dépôt

```bash
# Créer le dépôt local
mkdir portfolio
cd portfolio
git init

# Créer la structure
mkdir projects scripts docs

# Créer README
echo "# Mon Portfolio" > README.md

# Premier commit
git add .
git commit -m "Initial commit : portfolio"

# Créer sur GitHub et pousser
git remote add origin https://github.com/username/portfolio.git
git push -u origin main
```

---

## Projet 2 : Projet collaboratif

### Objectif

Gérer un projet avec plusieurs contributeurs.

### Workflow

```bash
# 1. Cloner le dépôt
git clone https://github.com/team/projet.git
cd projet

# 2. Créer une branche
git checkout -b feature-ma-contribution

# 3. Travailler
# ... modifications ...

# 4. Commiter
git add .
git commit -m "feat: Ajout nouvelle fonctionnalité"

# 5. Synchroniser avec main
git fetch origin
git rebase origin/main

# 6. Pousser
git push -u origin feature-ma-contribution

# 7. Créer Pull Request sur GitHub
```

### Gestion des conflits

```bash
# Si conflit après rebase
# 1. Résoudre le conflit
# 2. Ajouter les fichiers
git add .

# 3. Continuer le rebase
git rebase --continue

# 4. Pousser (force nécessaire après rebase)
git push --force-with-lease
```

---

## Projet 3 : Scripts Python versionnés

### Objectif

Versionner des scripts Python pour l'analyse de données.

### Structure

```
data-scripts/
├── README.md
├── .gitignore
├── requirements.txt
├── src/
│   ├── data_loader.py
│   ├── analyzer.py
│   └── visualizer.py
├── notebooks/
│   └── analysis.ipynb
└── data/
    └── .gitkeep
```

### .gitignore

```
__pycache__/
*.pyc
venv/
.env
*.csv
*.xlsx
data/
.ipynb_checkpoints
```

### Workflow

```bash
# Initialiser
git init
git add .
git commit -m "Initial commit : scripts d'analyse"

# Créer une branche pour nouvelle fonctionnalité
git checkout -b feature-nouvelle-analyse

# Développer
# ... code ...

# Commiter
git add src/analyzer.py
git commit -m "feat: Ajout analyse statistique avancée"

# Fusionner
git checkout main
git merge feature-nouvelle-analyse

# Taguer une version
git tag -a v1.0.0 -m "Version 1.0.0"
git push origin main --tags
```

---

## Projet 4 : Documentation de projet

### Objectif

Créer une documentation complète versionnée.

### Structure

```
project-docs/
├── README.md
├── docs/
│   ├── installation.md
│   ├── usage.md
│   ├── api.md
│   └── examples.md
└── CHANGELOG.md
```

### README.md complet

```markdown
# Nom du Projet

Description détaillée du projet.

## Table des matières

- [Installation](#installation)
- [Usage](#usage)
- [Documentation](#documentation)
- [Contribution](#contribution)

## Installation

\`\`\`bash
pip install -r requirements.txt
\`\`\`

## Usage

\`\`\`python
from project import fonction
resultat = fonction()
\`\`\`

## Documentation

Voir [docs/](docs/) pour la documentation complète.

## Contribution

Les contributions sont les bienvenues !

## License

MIT
```

### Workflow de documentation

```bash
# Créer une branche pour documentation
git checkout -b docs/ajout-guide-usage

# Ajouter la documentation
# ... écrire docs/usage.md ...

# Commiter
git add docs/usage.md
git commit -m "docs: Ajout guide d'utilisation"

# Pousser et créer PR
git push -u origin docs/ajout-guide-usage
```

---

## Exemples de projets portfolio

### Projet Data Analysis

```bash
# Structure
data-analysis-project/
├── README.md
├── data/
│   └── .gitkeep
├── notebooks/
│   └── analysis.ipynb
├── src/
│   └── analysis.py
└── results/
    └── .gitkeep
```

### Projet ETL Pipeline

```bash
# Structure
etl-pipeline/
├── README.md
├── src/
│   ├── extract.py
│   ├── transform.py
│   └── load.py
├── tests/
│   └── test_pipeline.py
└── config/
    └── config.yaml.example
```

---

## 📊 Points clés à retenir

1. **Portfolio** : Présenter vos projets
2. **Collaboration** : Workflow structuré
3. **Versioning** : Gérer les versions
4. **Documentation** : Essentielle
5. **GitHub** : Plateforme professionnelle

## 🔗 Ressources

- [GitHub Guides](https://guides.github.com/)
- [Git Documentation](https://git-scm.com/doc)
- [GitHub Student Pack](https://education.github.com/pack)

---

**Félicitations !** Vous avez terminé la formation Git. Vous pouvez maintenant gérer vos projets efficacement avec Git et GitHub.

