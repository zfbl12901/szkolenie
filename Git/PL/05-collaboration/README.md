# 5. Współpraca z Git

## 🎯 Cele

- Fork i Pull Requests
- Issues i Projects
- Code Review
- Workflow w zespole
- Dobre praktyki

## 📋 Spis treści

1. [Fork](#fork)
2. [Pull Requests](#pull-requests)
3. [Issues](#issues)
4. [Code Review](#code-review)
5. [Workflow w zespole](#workflow-w-zespole)

---

## Fork

### Czym jest Fork?

**Fork** = Kopia repozytorium w Twoim koncie

- **Kompletna kopia** : Wszystkie pliki i historia
- **Niezależna** : Zmiany bez wpływu na oryginał
- **Wkład** : Proponować zmiany przez PR

### Forkować repozytorium

**Na GitHub:**

1. Przejść do repozytorium
2. Kliknąć "Fork"
3. Wybrać swoje konto
4. Repozytorium jest skopiowane

**Klonować swój fork:**

```bash
# Klonować swój fork
git clone https://github.com/twoj-username/repo.git

# Dodać oryginał jako upstream
git remote add upstream https://github.com/original-owner/repo.git
```

---

## Pull Requests

### Utworzyć Pull Request

**Krok 1 : Utworzyć gałąź**

```bash
git checkout -b feature-moj-wklad
```

**Krok 2 : Wprowadzić modyfikacje**

```bash
# ... modyfikacje ...
git add .
git commit -m "Dodanie nowej funkcji"
```

**Krok 3 : Wypchnąć gałąź**

```bash
git push -u origin feature-moj-wklad
```

**Krok 4 : Utworzyć PR na GitHub**

1. Przejść do swojego forka
2. Kliknąć "Compare & pull request"
3. Wypełnić formularz
4. Kliknąć "Create pull request"

---

## Issues

### Utworzyć Issue

**Na GitHub:**

1. Przejść do repozytorium
2. Kliknąć "Issues"
3. Kliknąć "New Issue"
4. Wypełnić formularz

### Typy Issues

**Bug Report:**
- Opis błędu
- Kroki do reprodukcji
- Oczekiwane zachowanie

**Feature Request:**
- Opis funkcji
- Przypadki użycia

---

## Code Review

### Proces Review

1. **Utworzyć PR** : Z jasnym opisem
2. **Czekać na review** : Maintainerzy sprawdzają
3. **Poprawić** : Jeśli wymagane
4. **Zatwierdzić** : Po walidacji
5. **Połączyć** : Przez maintainera

---

## Workflow w zespole

### Standardowy workflow

```bash
# 1. Pobrać ostatnie zmiany
git pull origin main

# 2. Utworzyć gałąź
git checkout -b feature-nowa-funkcja

# 3. Pracować
# ... modyfikacje ...

# 4. Commitować regularnie
git add .
git commit -m "Jasna wiadomość"

# 5. Wypchnąć gałąź
git push -u origin feature-nowa-funkcja

# 6. Utworzyć PR
# Na GitHub/GitLab

# 7. Po połączeniu, wyczyścić
git checkout main
git pull origin main
git branch -d feature-nowa-funkcja
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **Fork** : Kopia repozytorium
2. **Pull Request** : Proponować zmiany
3. **Issues** : Śledzić problemy
4. **Code Review** : Weryfikacja kodu
5. **Workflow** : Ustrukturyzowany proces

## 🔗 Następny moduł

Przejdź do modułu [6. Funkcje zaawansowane](./06-advanced/README.md), aby pogłębić.

