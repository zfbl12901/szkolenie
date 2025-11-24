# 8. Git Practical Projects

## 🎯 Objectives

- Create a GitHub portfolio
- Manage a collaborative project
- Version Python scripts
- Document a project
- Portfolio projects

## 📋 Table of Contents

1. [Project 1 : GitHub Portfolio](#project-1--github-portfolio)
2. [Project 2 : Collaborative Project](#project-2--collaborative-project)
3. [Project 3 : Versioned Python Scripts](#project-3--versioned-python-scripts)
4. [Project 4 : Project Documentation](#project-4--project-documentation)

---

## Project 1 : GitHub Portfolio

### Objective

Create a professional portfolio on GitHub.

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

### Create Repository

```bash
# Create local repository
mkdir portfolio
cd portfolio
git init

# Create structure
mkdir projects scripts docs

# Create README
echo "# My Portfolio" > README.md

# First commit
git add .
git commit -m "Initial commit: portfolio"

# Create on GitHub and push
git remote add origin https://github.com/username/portfolio.git
git push -u origin main
```

---

## Project 2 : Collaborative Project

### Workflow

```bash
# 1. Clone repository
git clone https://github.com/team/project.git
cd project

# 2. Create branch
git checkout -b feature-my-contribution

# 3. Work
# ... modifications ...

# 4. Commit
git add .
git commit -m "feat: Add new feature"

# 5. Synchronize with main
git fetch origin
git rebase origin/main

# 6. Push
git push -u origin feature-my-contribution

# 7. Create Pull Request on GitHub
```

---

## Project 3 : Versioned Python Scripts

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
└── data/
    └── .gitkeep
```

### Workflow

```bash
# Initialize
git init
git add .
git commit -m "Initial commit: analysis scripts"

# Create branch for new feature
git checkout -b feature-new-analysis

# Develop
# ... code ...

# Commit
git add src/analyzer.py
git commit -m "feat: Add advanced statistical analysis"

# Merge
git checkout main
git merge feature-new-analysis

# Tag a version
git tag -a v1.0.0 -m "Version 1.0.0"
git push origin main --tags
```

---

## Project 4 : Project Documentation

### Structure

```
project-docs/
├── README.md
├── docs/
│   ├── installation.md
│   ├── usage.md
│   └── api.md
└── CHANGELOG.md
```

### Documentation Workflow

```bash
# Create branch for documentation
git checkout -b docs/add-usage-guide

# Add documentation
# ... write docs/usage.md ...

# Commit
git add docs/usage.md
git commit -m "docs: Add usage guide"

# Push and create PR
git push -u origin docs/add-usage-guide
```

---

## 📊 Key Takeaways

1. **Portfolio** : Showcase your projects
2. **Collaboration** : Structured workflow
3. **Versioning** : Manage versions
4. **Documentation** : Essential
5. **GitHub** : Professional platform

## 🔗 Resources

- [GitHub Guides](https://guides.github.com/)
- [Git Documentation](https://git-scm.com/doc)

---

**Congratulations!** You have completed the Git training. You can now manage your projects efficiently with Git and GitHub.

