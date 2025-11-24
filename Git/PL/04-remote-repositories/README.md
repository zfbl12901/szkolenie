# 4. Zdalne repozytoria Git

## 🎯 Cele

- Zrozumieć zdalne repozytoria
- Pracować z GitHub/GitLab
- Klonować repozytoria
- Push i Pull
- Synchronizacja

## 📋 Spis treści

1. [Wprowadzenie do zdalnych repozytoriów](#wprowadzenie-do-zdalnych-repozytoriów)
2. [GitHub i GitLab](#github-i-gitlab)
3. [Klonować repozytorium](#klonować-repozytorium)
4. [Push i Pull](#push-i-pull)
5. [Synchronizacja](#synchronizacja)

---

## Wprowadzenie do zdalnych repozytoriów

### Czym jest zdalne repozytorium?

**Zdalne repozytorium** = Kopia repozytorium na serwerze

- **GitHub** : Popularna usługa
- **GitLab** : Alternatywa open-source
- **Backup** : Kopia zapasowa online

---

## GitHub i GitLab

### Utworzyć konto GitHub

1. Przejść do: https://github.com
2. Kliknąć "Sign up"
3. Wypełnić formularz
4. Zweryfikować email

### Utworzyć repozytorium GitHub

1. Kliknąć "New repository"
2. Nazwać repozytorium
3. Wybrać public/private
4. Kliknąć "Create repository"

---

## Klonować repozytorium

### Klonować z GitHub

```bash
# Klonować z HTTPS
git clone https://github.com/username/repo.git

# Klonować z SSH
git clone git@github.com:username/repo.git
```

---

## Push i Pull

### Dodać remote

```bash
# Dodać remote
git remote add origin https://github.com/username/repo.git

# Zobaczyć remotes
git remote -v
```

### Push (wysłać)

```bash
# Pierwszy push
git push -u origin main

# Kolejne pushy
git push
```

### Pull (pobrać)

```bash
# Pobrać i połączyć
git pull

# Pobrać tylko
git fetch

# Połączyć po fetch
git merge origin/main
```

---

## Synchronizacja

### Podstawowy workflow

```bash
# 1. Pobrać ostatnie zmiany
git pull

# 2. Pracować lokalnie
# ... modyfikacje ...

# 3. Dodać i commitować
git add .
git commit -m "Modyfikacje"

# 4. Wysłać
git push
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **Remote** : Repozytorium na serwerze
2. **git clone** : Kopiować repozytorium
3. **git push** : Wysyłać zmiany
4. **git pull** : Pobierać zmiany
5. **Synchronizacja** : Pull przed Push

## 🔗 Następny moduł

Przejdź do modułu [5. Współpraca](./05-collaboration/README.md), aby nauczyć się współpracy.

