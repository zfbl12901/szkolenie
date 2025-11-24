# 2. Podstawowe polecenia Git

## 🎯 Cele

- Tworzyć i zarządzać repozytorium
- Dodawać i commitować pliki
- Widzieć historię
- Cofać zmiany
- Ignorować pliki

## 📋 Spis treści

1. [Utworzyć repozytorium](#utworzyć-repozytorium)
2. [Dodawać i commitować](#dodawać-i-commitować)
3. [Widzieć historię](#widzieć-historię)
4. [Cofać zmiany](#cofać-zmiany)
5. [.gitignore](#gitignore)

---

## Utworzyć repozytorium

### Zainicjalizować lokalne repozytorium

```bash
# Utworzyć nowe repozytorium
mkdir moj-projekt
cd moj-projekt
git init

# Sprawdzić
ls -la  # Zobaczyć .git
```

### Klonować istniejące repozytorium

```bash
# Klonować z GitHub
git clone https://github.com/username/repo.git

# Klonować do konkretnego folderu
git clone https://github.com/username/repo.git moj-folder

# Klonować z SSH
git clone git@github.com:username/repo.git
```

---

## Dodawać i commitować

### Podstawowy workflow

```bash
# 1. Zobaczyć status
git status

# 2. Dodawać pliki
git add plik.py
git add .  # Wszystkie pliki

# 3. Commitować
git commit -m "Wiadomość commit"
```

### Wiadomości commit

**Dobry format:**
```
Type: Krótki opis (max 50 znaków)

Szczegółowy opis jeśli potrzebny
```

**Przykłady:**
```
feat: Dodanie funkcji analizy danych
fix: Poprawka błędu obliczeń
docs: Aktualizacja README
```

---

## Widzieć historię

### git log

```bash
# Pełna historia
git log

# Kompaktowa historia
git log --oneline

# Historia z wykresem
git log --graph --oneline --all

# Ograniczyć liczbę
git log -5  # Ostatnie 5 commitów
```

---

## Cofać zmiany

### Cofać w working directory

```bash
# Cofać modyfikacje pliku
git restore plik.py

# Cofać wszystkie modyfikacje
git restore .
```

### Cofać w staging

```bash
# Usunąć ze staging
git restore --staged plik.py
```

### Modyfikować ostatni commit

```bash
# Modyfikować wiadomość
git commit --amend -m "Nowa wiadomość"

# Dodać zapomniane pliki
git add zapomniany_plik.py
git commit --amend --no-edit
```

---

## .gitignore

### Utworzyć .gitignore

```
# Python
__pycache__/
*.py[cod]
venv/
env/

# Jupyter
.ipynb_checkpoints
*.ipynb

# Dane
*.csv
*.xlsx
data/

# IDE
.vscode/
.idea/

# Sekrety
.env
*.key
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **git add** : Dodawać do staging
2. **git commit** : Tworzyć commit
3. **git log** : Widzieć historię
4. **git status** : Widzieć status
5. **.gitignore** : Wykluczać pliki

## 🔗 Następny moduł

Przejdź do modułu [3. Gałęzie](./03-branching/README.md), aby nauczyć się zarządzania gałęziami.

