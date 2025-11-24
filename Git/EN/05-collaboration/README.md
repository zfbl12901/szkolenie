# 5. Git Collaboration

## 🎯 Objectives

- Fork and Pull Requests
- Issues and Projects
- Code Review
- Team workflow
- Best practices

## 📋 Table of Contents

1. [Fork](#fork)
2. [Pull Requests](#pull-requests)
3. [Issues](#issues)
4. [Code Review](#code-review)
5. [Team Workflow](#team-workflow)

---

## Fork

### What is a Fork?

**Fork** = Copy of a repository in your account

- **Complete copy** : All files and history
- **Independent** : Changes without affecting original
- **Contribution** : Propose changes via PR

### Fork a Repository

**On GitHub:**

1. Go to repository
2. Click "Fork"
3. Choose your account
4. Repository is copied

**Clone your fork:**

```bash
# Clone your fork
git clone https://github.com/your-username/repo.git

# Add original as upstream
git remote add upstream https://github.com/original-owner/repo.git
```

---

## Pull Requests

### Create a Pull Request

**Step 1 : Create a branch**

```bash
git checkout -b feature-my-contribution
```

**Step 2 : Make modifications**

```bash
# ... modifications ...
git add .
git commit -m "Add new feature"
```

**Step 3 : Push branch**

```bash
git push -u origin feature-my-contribution
```

**Step 4 : Create PR on GitHub**

1. Go to your fork
2. Click "Compare & pull request"
3. Fill form
4. Click "Create pull request"

---

## Issues

### Create an Issue

**On GitHub:**

1. Go to repository
2. Click "Issues"
3. Click "New Issue"
4. Fill form

### Issue Types

**Bug Report:**
- Bug description
- Steps to reproduce
- Expected behavior

**Feature Request:**
- Feature description
- Use cases

---

## Code Review

### Review Process

1. **Create PR** : With clear description
2. **Wait for review** : Maintainers check
3. **Fix** : If requested
4. **Approve** : Once validated
5. **Merge** : By maintainer

---

## Team Workflow

### Standard Workflow

```bash
# 1. Retrieve latest changes
git pull origin main

# 2. Create a branch
git checkout -b feature-new-function

# 3. Work
# ... modifications ...

# 4. Commit regularly
git add .
git commit -m "Clear message"

# 5. Push branch
git push -u origin feature-new-function

# 6. Create PR
# On GitHub/GitLab

# 7. After merge, clean up
git checkout main
git pull origin main
git branch -d feature-new-function
```

---

## 📊 Key Takeaways

1. **Fork** : Copy of a repository
2. **Pull Request** : Propose changes
3. **Issues** : Track problems
4. **Code Review** : Code verification
5. **Workflow** : Structured process

## 🔗 Next Module

Proceed to module [6. Advanced Features](./06-advanced/README.md) to deepen.

