# 2. Git Basic Commands

## 🎯 Objectives

- Create and manage a repository
- Add and commit files
- View history
- Undo changes
- Ignore files

## 📋 Table of Contents

1. [Create a Repository](#create-a-repository)
2. [Add and Commit](#add-and-commit)
3. [View History](#view-history)
4. [Undo Changes](#undo-changes)
5. [.gitignore](#gitignore)

---

## Create a Repository

### Initialize a Local Repository

```bash
# Create a new repository
mkdir my-project
cd my-project
git init

# Verify
ls -la  # See .git
```

### Clone an Existing Repository

```bash
# Clone from GitHub
git clone https://github.com/username/repo.git

# Clone into specific folder
git clone https://github.com/username/repo.git my-folder

# Clone with SSH
git clone git@github.com:username/repo.git
```

---

## Add and Commit

### Basic Workflow

```bash
# 1. See status
git status

# 2. Add files
git add file.py
git add .  # All files

# 3. Commit
git commit -m "Commit message"
```

### Commit Messages

**Good format:**
```
Type: Short description (max 50 characters)

Detailed description if needed
```

**Examples:**
```
feat: Add data analysis function
fix: Fix calculation bug
docs: Update README
```

---

## View History

### git log

```bash
# Complete history
git log

# Compact history
git log --oneline

# History with graph
git log --graph --oneline --all

# Limit number
git log -5  # Last 5 commits
```

---

## Undo Changes

### Undo in Working Directory

```bash
# Undo file modifications
git restore file.py

# Undo all modifications
git restore .
```

### Undo in Staging

```bash
# Remove from staging
git restore --staged file.py
```

### Modify Last Commit

```bash
# Modify message
git commit --amend -m "New message"

# Add forgotten files
git add forgotten_file.py
git commit --amend --no-edit
```

---

## .gitignore

### Create .gitignore

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

## 📊 Key Takeaways

1. **git add** : Add to staging
2. **git commit** : Create commit
3. **git log** : View history
4. **git status** : View status
5. **.gitignore** : Exclude files

## 🔗 Next Module

Proceed to module [3. Branching](./03-branching/README.md) to learn branch management.

