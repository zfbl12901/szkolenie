# 6. Git Advanced Features

## 🎯 Objectives

- Use Stash
- Understand Rebase
- Manage Tags
- Use Hooks
- Advanced commands

## 📋 Table of Contents

1. [Stash](#stash)
2. [Rebase](#rebase)
3. [Tags](#tags)
4. [Hooks](#hooks)
5. [Advanced Commands](#advanced-commands)

---

## Stash

### What is Stash?

**Stash** = Temporarily save changes

- **Temporary** : Uncommitted changes
- **Quick** : Switch branches quickly
- **Recoverable** : Retrieve later

### Use Stash

```bash
# Save changes
git stash

# With message
git stash save "Descriptive message"

# Apply last stash
git stash apply

# Apply and remove
git stash pop

# List stashes
git stash list
```

---

## Rebase

### What is Rebase?

**Rebase** = Reapply commits on another base

- **Linear history** : Cleaner
- **Rewriting** : Modifies history
- **Caution** : Don't rebase on shared branches

### Simple Rebase

```bash
# Rebase on main
git checkout feature-branch
git rebase main

# Interactive rebase
git rebase -i HEAD~3
```

---

## Tags

### What is a Tag?

**Tag** = Pointer to specific commit

- **Version** : Mark versions
- **Release** : Release points
- **Reference** : Stable reference

### Create a Tag

```bash
# Lightweight tag
git tag v1.0.0

# Annotated tag (recommended)
git tag -a v1.0.0 -m "Version 1.0.0"

# Push tag
git push origin v1.0.0
```

---

## Hooks

### What is a Hook?

**Hook** = Script executed at certain events

- **Automation** : Execute actions
- **Validation** : Check before commit
- **Notification** : Notify after push

### Example Hook

**`.git/hooks/pre-commit`:**

```bash
#!/bin/bash
# Run tests before commit
python -m pytest tests/

if [ $? -ne 0 ]; then
    echo "Tests failed, commit cancelled"
    exit 1
fi
```

---

## Advanced Commands

### Cherry-pick

```bash
# Apply specific commit
git cherry-pick <hash>
```

### Reflog

```bash
# See action history
git reflog

# Recover lost commit
git checkout <hash>
```

---

## 📊 Key Takeaways

1. **Stash** : Temporarily save
2. **Rebase** : Rewrite history
3. **Tags** : Mark versions
4. **Hooks** : Automate actions
5. **Advanced commands** : Powerful tools

## 🔗 Next Module

Proceed to module [7. Best Practices](./07-best-practices/README.md) for best practices.

