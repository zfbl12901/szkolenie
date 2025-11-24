# 8. Projekty praktyczne Git

## 🎯 Cele

- Utworzyć portfolio GitHub
- Zarządzać projektem współpracującym
- Wersjonować skrypty Python
- Dokumentować projekt
- Projekty do portfolio

## 📋 Spis treści

1. [Projekt 1 : Portfolio GitHub](#projekt-1--portfolio-github)
2. [Projekt 2 : Projekt współpracujący](#projekt-2--projekt-współpracujący)
3. [Projekt 3 : Wersjonowane skrypty Python](#projekt-3--wersjonowane-skrypty-python)
4. [Projekt 4 : Dokumentacja projektu](#projekt-4--dokumentacja-projektu)

---

## Projekt 1 : Portfolio GitHub

### Cel

Utworzyć profesjonalne portfolio na GitHub.

### Struktura

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

### Utworzyć repozytorium

```bash
# Utworzyć lokalne repozytorium
mkdir portfolio
cd portfolio
git init

# Utworzyć strukturę
mkdir projects scripts docs

# Utworzyć README
echo "# Moje Portfolio" > README.md

# Pierwszy commit
git add .
git commit -m "Initial commit: portfolio"

# Utworzyć na GitHub i wypchnąć
git remote add origin https://github.com/username/portfolio.git
git push -u origin main
```

---

## Projekt 2 : Projekt współpracujący

### Workflow

```bash
# 1. Klonować repozytorium
git clone https://github.com/team/projekt.git
cd projekt

# 2. Utworzyć gałąź
git checkout -b feature-moj-wklad

# 3. Pracować
# ... modyfikacje ...

# 4. Commitować
git add .
git commit -m "feat: Dodanie nowej funkcji"

# 5. Synchronizować z main
git fetch origin
git rebase origin/main

# 6. Wypchnąć
git push -u origin feature-moj-wklad

# 7. Utworzyć Pull Request na GitHub
```

---

## Projekt 3 : Wersjonowane skrypty Python

### Struktura

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
# Zainicjalizować
git init
git add .
git commit -m "Initial commit: skrypty analizy"

# Utworzyć gałąź dla nowej funkcji
git checkout -b feature-nowa-analiza

# Rozwijać
# ... kod ...

# Commitować
git add src/analyzer.py
git commit -m "feat: Dodanie zaawansowanej analizy statystycznej"

# Łączyć
git checkout main
git merge feature-nowa-analiza

# Oznaczyć wersję
git tag -a v1.0.0 -m "Wersja 1.0.0"
git push origin main --tags
```

---

## Projekt 4 : Dokumentacja projektu

### Struktura

```
project-docs/
├── README.md
├── docs/
│   ├── installation.md
│   ├── usage.md
│   └── api.md
└── CHANGELOG.md
```

### Workflow dokumentacji

```bash
# Utworzyć gałąź dla dokumentacji
git checkout -b docs/dodanie-guide-usage

# Dodać dokumentację
# ... pisać docs/usage.md ...

# Commitować
git add docs/usage.md
git commit -m "docs: Dodanie guide użycia"

# Wypchnąć i utworzyć PR
git push -u origin docs/dodanie-guide-usage
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **Portfolio** : Prezentować projekty
2. **Współpraca** : Ustrukturyzowany workflow
3. **Wersjonowanie** : Zarządzać wersjami
4. **Dokumentacja** : Niezbędna
5. **GitHub** : Profesjonalna platforma

## 🔗 Zasoby

- [Przewodniki GitHub](https://guides.github.com/)
- [Dokumentacja Git](https://git-scm.com/doc)

---

**Gratulacje !** Ukończyłeś szkolenie Git. Możesz teraz zarządzać projektami efektywnie z Git i GitHub.

