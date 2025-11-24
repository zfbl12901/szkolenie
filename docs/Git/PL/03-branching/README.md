# 3. Gałęzie Git

## 🎯 Cele

- Zrozumieć gałęzie
- Tworzyć i zarządzać gałęziami
- Łączyć gałęzie
- Rozwiązywać konflikty
- Workflow z gałęziami

## 📋 Spis treści

1. [Wprowadzenie do gałęzi](#wprowadzenie-do-gałęzi)
2. [Tworzyć gałęzie](#tworzyć-gałęzie)
3. [Łączyć gałęzie](#łączyć-gałęzie)
4. [Rozwiązywać konflikty](#rozwiązywać-konflikty)
5. [Workflow](#workflow)

---

## Wprowadzenie do gałęzi

### Czym jest gałąź?

**Gałąź** = Niezależna linia rozwoju

- **Izolacja** : Praca izolowana
- **Równoległa** : Wiele gałęzi jednocześnie
- **Łączenie** : Łączyć zmiany
- **main/master** : Główna gałąź

---

## Tworzyć gałęzie

### Utworzyć nową gałąź

```bash
# Utworzyć gałąź
git branch feature-analyse

# Utworzyć i przełączyć
git checkout -b feature-analyse

# Nowa składnia (Git 2.23+)
git switch -c feature-analyse
```

### Przełączać między gałęziami

```bash
# Przełączyć na gałąź
git checkout feature-analyse

# Nowa składnia
git switch feature-analyse

# Wrócić do main
git checkout main
```

---

## Łączyć gałęzie

### Łączenie

```bash
# Przełączyć na main
git checkout main

# Łączyć gałąź
git merge feature-analyse
```

---

## Rozwiązywać konflikty

### Kiedy występują konflikty?

- **Ta sama linia zmodyfikowana** : W dwóch różnych gałęziach
- **Plik usunięty** : W jednej gałęzi, zmodyfikowany w drugiej

### Rozwiązać konflikt

**Krok 1 : Zidentyfikować konflikt**

```bash
git status
```

**Krok 2 : Otworzyć plik**

```python
<<<<<<< HEAD
print("Wersja main")
=======
print("Wersja feature")
>>>>>>> feature-analyse
```

**Krok 3 : Rozwiązać ręcznie**

```python
print("Wersja połączona")
```

**Krok 4 : Oznaczyć jako rozwiązane**

```bash
git add plik.py
git commit
```

---

## Workflow

### Prosty workflow

```bash
# 1. Utworzyć gałąź dla funkcji
git checkout -b feature-nowa-funkcja

# 2. Pracować na gałęzi
# ... modyfikacje ...

# 3. Commitować
git add .
git commit -m "Dodanie nowej funkcji"

# 4. Łączyć z main
git checkout main
git merge feature-nowa-funkcja

# 5. Usunąć gałąź
git branch -d feature-nowa-funkcja
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **Gałęzie** : Izolowane linie rozwoju
2. **git branch** : Tworzyć/zarądzać gałęziami
3. **git merge** : Łączyć gałęzie
4. **Konflikty** : Rozwiązywać ręcznie
5. **Workflow** : Jedna gałąź na funkcję

## 🔗 Następny moduł

Przejdź do modułu [4. Zdalne repozytoria](./04-remote-repositories/README.md), aby pracować z GitHub/GitLab.

