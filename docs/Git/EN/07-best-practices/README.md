# 7. Git Best Practices

## 🎯 Objectives

- Effective commit messages
- Project structure
- Complete .gitignore
- Documentation
- Optimal workflow

## 📋 Table of Contents

1. [Commit Messages](#commit-messages)
2. [Project Structure](#project-structure)
3. [.gitignore](#gitignore)
4. [Documentation](#documentation)
5. [Workflow](#workflow)

---

## Commit Messages

### Recommended Format

```
Type: Short description (max 50 characters)

Detailed description if needed (72 characters per line)

- Point 1
- Point 2
```

### Commit Types

- **feat** : New feature
- **fix** : Bug fix
- **docs** : Documentation
- **style** : Formatting
- **refactor** : Refactoring
- **test** : Tests
- **chore** : Maintenance tasks

---

## Project Structure

### Recommended Structure

```
my-project/
├── README.md
├── .gitignore
├── LICENSE
├── requirements.txt
├── src/
│   ├── __init__.py
│   └── main.py
├── tests/
│   └── test_main.py
└── docs/
    └── guide.md
```

### README.md

**Essential content:**

```markdown
# Project Name

Short project description.

## Installation

\`\`\`bash
pip install -r requirements.txt
\`\`\`

## Usage

\`\`\`python
from src.main import function
function()
\`\`\`
```

---

## .gitignore

### Complete .gitignore for Python

```
# Python
__pycache__/
*.py[cod]
venv/
env/

# Jupyter
.ipynb_checkpoints
*.ipynb

# Data
*.csv
*.xlsx
data/

# IDE
.vscode/
.idea/

# Secrets
.env
*.key
```

---

## Documentation

### Code Documentation

**Python Docstrings:**

```python
def analyze_data(file):
    """
    Analyze a CSV data file.
    
    Args:
        file (str): Path to CSV file
        
    Returns:
        dict: Dictionary with statistics
    """
    # Code...
```

---

## Workflow

### Recommended Workflow

1. **Create a branch** : For each feature
2. **Commit regularly** : Small frequent commits
3. **Test** : Before pushing
4. **Pull Request** : For review
5. **Merge** : After approval

### Golden Rules

- **One commit = One logical change**
- **Clear and descriptive messages**
- **Test before pushing**
- **Never force push on main**
- **Synchronize regularly**

---

## 📊 Key Takeaways

1. **Messages** : Clear and structured
2. **Structure** : Organized and logical
3. **.gitignore** : Complete and adapted
4. **Documentation** : README and docstrings
5. **Workflow** : Regular and consistent

## 🔗 Next Module

Proceed to module [8. Practical Projects](./08-projets/README.md) to create complete projects.

