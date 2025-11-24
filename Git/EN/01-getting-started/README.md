# 1. Git Getting Started

## 🎯 Objectives

- Understand Git and version control
- Install Git
- Configure Git
- Create your first repository
- Understand basic concepts

## 📋 Table of Contents

1. [Introduction to Git](#introduction-to-git)
2. [Installation](#installation)
3. [Configuration](#configuration)
4. [First Repository](#first-repository)
5. [Basic Concepts](#basic-concepts)

---

## Introduction to Git

### What is Git?

**Git** = Distributed version control system

- **Versioning** : Tracks file changes
- **Distributed** : Each developer has a complete copy
- **Collaboration** : Facilitates team work
- **History** : Preserves complete history

### Why Git for Data Analyst?

- **Scripts** : Version your Python/R scripts
- **Projects** : Manage your portfolio projects
- **Collaboration** : Work in teams
- **Backup** : Backup online (GitHub)
- **Documentation** : Version your documentation

---

## Installation

### Windows

1. Go to: https://git-scm.com/download/win
2. Download the installer
3. Run the installer
4. Accept default options

### Linux

**Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install git
git --version
```

### macOS

**With Homebrew:**
```bash
brew install git
git --version
```

---

## Configuration

### Global Configuration

```bash
# Configure your name
git config --global user.name "Your Name"

# Configure your email
git config --global user.email "your.email@example.com"

# Configure default editor
git config --global core.editor "code --wait"  # VS Code
```

### Verify Configuration

```bash
# See all configuration
git config --list

# See specific configuration
git config user.name
```

---

## First Repository

### Create a New Repository

```bash
# Create a directory
mkdir my-project
cd my-project

# Initialize Git
git init

# Verify
ls -la  # See .git folder
```

### First Commit

```bash
# Create a file
echo "# My Project" > README.md

# See status
git status

# Add the file
git add README.md

# Commit
git commit -m "First commit: add README"
```

---

## Basic Concepts

### Repository

**Repository** = Folder with Git history

- **Local** : On your machine
- **Remote** : On GitHub/GitLab
- **.git** : Hidden folder containing history

### Commit

**Commit** = Point in history

- **Snapshot** : Capture of file state
- **Message** : Description of changes
- **Author** : Name and email
- **Hash** : Unique identifier (SHA-1)

### Branch

**Branch** = Development line

- **main/master** : Main branch
- **Other branches** : For new features
- **Isolation** : Isolated work

---

## 📊 Key Takeaways

1. **Git** = Distributed version control
2. **Repository** = Folder with history
3. **Commit** = Point in history
4. **Branch** = Development line
5. **Staging** = Preparation area

## 🔗 Next Module

Proceed to module [2. Basic Commands](./02-basic-commands/README.md) to master essential commands.

