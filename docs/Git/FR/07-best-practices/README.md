# 7. Bonnes pratiques Git

## 🎯 Objectifs

- Messages de commit efficaces
- Structure de projet
- .gitignore complet
- Documentation
- Workflow optimal

## 📋 Table des matières

1. [Messages de commit](#messages-de-commit)
2. [Structure de projet](#structure-de-projet)
3. [.gitignore](#gitignore)
4. [Documentation](#documentation)
5. [Workflow](#workflow)

---

## Messages de commit

### Format recommandé

```
Type : Description courte (50 caractères max)

Description détaillée si nécessaire (72 caractères par ligne)

- Point 1
- Point 2
```

### Types de commit

- **feat** : Nouvelle fonctionnalité
- **fix** : Correction de bug
- **docs** : Documentation
- **style** : Formatage (pas de changement de code)
- **refactor** : Refactorisation
- **test** : Tests
- **chore** : Tâches de maintenance

### Exemples

**Bon :**
```
feat: Ajout fonction analyse de données

Implémentation d'une fonction pour analyser les données CSV
avec support des colonnes multiples.

- Lecture des fichiers CSV
- Calcul des statistiques
- Export des résultats
```

**Mauvais :**
```
modifications
```

---

## Structure de projet

### Structure recommandée

```
mon-projet/
├── README.md
├── .gitignore
├── LICENSE
├── requirements.txt
├── src/
│   ├── __init__.py
│   └── main.py
├── tests/
│   └── test_main.py
├── docs/
│   └── guide.md
└── data/
    └── .gitkeep
```

### README.md

**Contenu essentiel :**

```markdown
# Nom du Projet

Description courte du projet.

## Installation

```bash
pip install -r requirements.txt
```

## Usage

```python
from src.main import fonction
fonction()
```

## Contribution

Les contributions sont les bienvenues !

## License

MIT
```

---

## .gitignore

### .gitignore complet pour Python

```
# Byte-compiled / optimized / DLL files
__pycache__/
*.py[cod]
*$py.class

# Virtual environments
venv/
env/
ENV/

# IDEs
.vscode/
.idea/
*.swp
*.swo

# Jupyter Notebook
.ipynb_checkpoints
*.ipynb

# Data files
*.csv
*.xlsx
*.parquet
data/
*.db
*.sqlite

# Secrets
.env
*.key
config.ini
secrets/

# OS
.DS_Store
Thumbs.db

# Logs
*.log
logs/
```

### .gitignore pour Data Science

```
# Data
data/
*.csv
*.xlsx
*.parquet
*.h5
*.hdf5

# Models
models/
*.pkl
*.joblib

# Notebooks (optionnel)
*.ipynb

# Results
results/
outputs/
```

---

## Documentation

### README.md

**Sections essentielles :**

1. **Titre et description**
2. **Installation**
3. **Usage**
4. **Exemples**
5. **Contribution**
6. **License**

### Documentation du code

**Docstrings Python :**

```python
def analyse_donnees(fichier):
    """
    Analyse un fichier de données CSV.
    
    Args:
        fichier (str): Chemin vers le fichier CSV
        
    Returns:
        dict: Dictionnaire avec les statistiques
        
    Example:
        >>> stats = analyse_donnees('data.csv')
        >>> print(stats['moyenne'])
    """
    # Code...
```

### CHANGELOG.md

```markdown
# Changelog

## [1.0.0] - 2024-01-15

### Added
- Fonction analyse de données
- Support CSV

### Fixed
- Bug dans le calcul de moyenne

### Changed
- Amélioration de la documentation
```

---

## Workflow

### Workflow recommandé

1. **Créer une branche** : Pour chaque fonctionnalité
2. **Commiter régulièrement** : Petits commits fréquents
3. **Tester** : Avant de pousser
4. **Pull Request** : Pour review
5. **Fusionner** : Après approbation

### Règles d'or

- **Un commit = Une modification logique**
- **Messages clairs et descriptifs**
- **Tester avant de pousser**
- **Ne jamais force push sur main**
- **Synchroniser régulièrement**

---

## Exemples pratiques

### Exemple 1 : Projet Python structuré

```
data-analysis/
├── README.md
├── .gitignore
├── requirements.txt
├── setup.py
├── src/
│   ├── __init__.py
│   ├── data_loader.py
│   └── analyzer.py
├── tests/
│   ├── __init__.py
│   └── test_analyzer.py
└── docs/
    └── guide.md
```

### Exemple 2 : Workflow de commit

```bash
# 1. Créer une branche
git checkout -b feature-analyse

# 2. Faire des modifications
# ... code ...

# 3. Tester
python -m pytest tests/

# 4. Commiter
git add src/analyzer.py
git commit -m "feat: Ajout fonction analyse statistique"

# 5. Pousser
git push -u origin feature-analyse

# 6. Créer PR
```

---

## 📊 Points clés à retenir

1. **Messages** : Clairs et structurés
2. **Structure** : Organisée et logique
3. **.gitignore** : Complet et adapté
4. **Documentation** : README et docstrings
5. **Workflow** : Régulier et cohérent

## 🔗 Prochain module

Passer au module [8. Projets pratiques](./08-projets/README.md) pour créer des projets complets.

