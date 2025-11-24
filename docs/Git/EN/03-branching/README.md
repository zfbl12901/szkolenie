# 3. Git Branching

## 🎯 Objectives

- Understand branches
- Create and manage branches
- Merge branches
- Resolve conflicts
- Branch workflow

## 📋 Table of Contents

1. [Introduction to Branches](#introduction-to-branches)
2. [Create Branches](#create-branches)
3. [Merge Branches](#merge-branches)
4. [Resolve Conflicts](#resolve-conflicts)
5. [Workflow](#workflow)

---

## Introduction to Branches

### What is a Branch?

**Branch** = Independent development line

- **Isolation** : Isolated work
- **Parallel** : Multiple branches at once
- **Merge** : Combine changes
- **main/master** : Main branch

---

## Create Branches

### Create a New Branch

```bash
# Create a branch
git branch feature-analysis

# Create and switch
git checkout -b feature-analysis

# New syntax (Git 2.23+)
git switch -c feature-analysis
```

### Switch Between Branches

```bash
# Switch to a branch
git checkout feature-analysis

# New syntax
git switch feature-analysis

# Return to main
git checkout main
```

---

## Merge Branches

### Merge

```bash
# Switch to main
git checkout main

# Merge the branch
git merge feature-analysis
```

---

## Resolve Conflicts

### When Conflicts Occur

- **Same line modified** : In two different branches
- **File deleted** : In one branch, modified in other

### Resolve a Conflict

**Step 1 : Identify conflict**

```bash
git status
```

**Step 2 : Open file**

```python
<<<<<<< HEAD
print("Main version")
=======
print("Feature version")
>>>>>>> feature-analysis
```

**Step 3 : Resolve manually**

```python
print("Combined version")
```

**Step 4 : Mark as resolved**

```bash
git add file.py
git commit
```

---

## Workflow

### Simple Workflow

```bash
# 1. Create branch for feature
git checkout -b feature-new-function

# 2. Work on branch
# ... modifications ...

# 3. Commit
git add .
git commit -m "Add new feature"

# 4. Merge into main
git checkout main
git merge feature-new-function

# 5. Delete branch
git branch -d feature-new-function
```

---

## 📊 Key Takeaways

1. **Branches** : Isolated development lines
2. **git branch** : Create/manage branches
3. **git merge** : Merge branches
4. **Conflicts** : Resolve manually
5. **Workflow** : One branch per feature

## 🔗 Next Module

Proceed to module [4. Remote Repositories](./04-remote-repositories/README.md) to work with GitHub/GitLab.

