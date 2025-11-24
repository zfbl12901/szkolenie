# 4. Git Remote Repositories

## 🎯 Objectives

- Understand remote repositories
- Work with GitHub/GitLab
- Clone repositories
- Push and Pull
- Synchronization

## 📋 Table of Contents

1. [Introduction to Remote Repositories](#introduction-to-remote-repositories)
2. [GitHub and GitLab](#github-and-gitlab)
3. [Clone a Repository](#clone-a-repository)
4. [Push and Pull](#push-and-pull)
5. [Synchronization](#synchronization)

---

## Introduction to Remote Repositories

### What is a Remote Repository?

**Remote Repository** = Copy of repository on a server

- **GitHub** : Popular service
- **GitLab** : Open-source alternative
- **Backup** : Online backup

---

## GitHub and GitLab

### Create GitHub Account

1. Go to: https://github.com
2. Click "Sign up"
3. Fill form
4. Verify email

### Create GitHub Repository

1. Click "New repository"
2. Name the repository
3. Choose public/private
4. Click "Create repository"

---

## Clone a Repository

### Clone from GitHub

```bash
# Clone with HTTPS
git clone https://github.com/username/repo.git

# Clone with SSH
git clone git@github.com:username/repo.git
```

---

## Push and Pull

### Add a Remote

```bash
# Add a remote
git remote add origin https://github.com/username/repo.git

# See remotes
git remote -v
```

### Push (Send)

```bash
# First push
git push -u origin main

# Subsequent pushes
git push
```

### Pull (Retrieve)

```bash
# Retrieve and merge
git pull

# Retrieve only
git fetch

# Merge after fetch
git merge origin/main
```

---

## Synchronization

### Basic Workflow

```bash
# 1. Retrieve latest changes
git pull

# 2. Work locally
# ... modifications ...

# 3. Add and commit
git add .
git commit -m "Modifications"

# 4. Send
git push
```

---

## 📊 Key Takeaways

1. **Remote** : Repository on server
2. **git clone** : Copy a repository
3. **git push** : Send changes
4. **git pull** : Retrieve changes
5. **Synchronization** : Pull before Push

## 🔗 Next Module

Proceed to module [5. Collaboration](./05-collaboration/README.md) to learn collaboration.

