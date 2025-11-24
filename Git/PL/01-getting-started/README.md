# 1. Rozpoczęcie z Git

## 🎯 Cele

- Zrozumieć Git i kontrolę wersji
- Zainstalować Git
- Skonfigurować Git
- Utworzyć pierwsze repozytorium
- Zrozumieć podstawowe koncepcje

## 📋 Spis treści

1. [Wprowadzenie do Git](#wprowadzenie-do-git)
2. [Instalacja](#instalacja)
3. [Konfiguracja](#konfiguracja)
4. [Pierwsze repozytorium](#pierwsze-repozytorium)
5. [Podstawowe koncepcje](#podstawowe-koncepcje)

---

## Wprowadzenie do Git

### Czym jest Git?

**Git** = Rozproszony system kontroli wersji

- **Wersjonowanie** : Śledzi zmiany plików
- **Rozproszony** : Każdy developer ma kompletną kopię
- **Współpraca** : Ułatwia pracę w zespole
- **Historia** : Zachowuje kompletną historię

### Dlaczego Git dla Data Analyst?

- **Skrypty** : Wersjonować skrypty Python/R
- **Projekty** : Zarządzać projektami portfolio
- **Współpraca** : Pracować w zespołach
- **Backup** : Tworzyć kopię zapasową online (GitHub)
- **Dokumentacja** : Wersjonować dokumentację

---

## Instalacja

### Windows

1. Przejść do: https://git-scm.com/download/win
2. Pobrać instalator
3. Uruchomić instalator
4. Zaakceptować opcje domyślne

### Linux

**Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install git
git --version
```

### macOS

**Z Homebrew:**
```bash
brew install git
git --version
```

---

## Konfiguracja

### Konfiguracja globalna

```bash
# Skonfigurować imię
git config --global user.name "Twoje Imię"

# Skonfigurować email
git config --global user.email "twoj.email@example.com"

# Skonfigurować edytor domyślny
git config --global core.editor "code --wait"  # VS Code
```

### Sprawdzić konfigurację

```bash
# Zobaczyć całą konfigurację
git config --list

# Zobaczyć konkretną konfigurację
git config user.name
```

---

## Pierwsze repozytorium

### Utworzyć nowe repozytorium

```bash
# Utworzyć katalog
mkdir moj-projekt
cd moj-projekt

# Zainicjalizować Git
git init

# Sprawdzić
ls -la  # Zobaczyć folder .git
```

### Pierwszy commit

```bash
# Utworzyć plik
echo "# Mój Projekt" > README.md

# Zobaczyć status
git status

# Dodać plik
git add README.md

# Commitować
git commit -m "Pierwszy commit: dodanie README"
```

---

## Podstawowe koncepcje

### Repozytorium

**Repozytorium** = Folder z historią Git

- **Lokalne** : Na Twojej maszynie
- **Zdalne** : Na GitHub/GitLab
- **.git** : Ukryty folder zawierający historię

### Commit

**Commit** = Punkt w historii

- **Snapshot** : Zapis stanu plików
- **Wiadomość** : Opis zmian
- **Autor** : Imię i email
- **Hash** : Unikalny identyfikator (SHA-1)

### Gałąź

**Gałąź** = Linia rozwoju

- **main/master** : Główna gałąź
- **Inne gałęzie** : Dla nowych funkcji
- **Izolacja** : Praca izolowana

---

## 📊 Kluczowe punkty do zapamiętania

1. **Git** = Rozproszona kontrola wersji
2. **Repozytorium** = Folder z historią
3. **Commit** = Punkt w historii
4. **Gałąź** = Linia rozwoju
5. **Staging** = Obszar przygotowania

## 🔗 Następny moduł

Przejdź do modułu [2. Podstawowe polecenia](./02-basic-commands/README.md), aby opanować podstawowe polecenia.

