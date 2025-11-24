# 6. Funkcje zaawansowane Git

## 🎯 Cele

- Używać Stash
- Zrozumieć Rebase
- Zarządzać Tagami
- Używać Hooków
- Polecenia zaawansowane

## 📋 Spis treści

1. [Stash](#stash)
2. [Rebase](#rebase)
3. [Tagi](#tagi)
4. [Hooki](#hooki)
5. [Polecenia zaawansowane](#polecenia-zaawansowane)

---

## Stash

### Czym jest Stash?

**Stash** = Tymczasowo zapisać zmiany

- **Tymczasowy** : Niecommitowane zmiany
- **Szybki** : Szybko przełączać gałęzie
- **Odzyskiwalny** : Odzyskać później

### Używać Stash

```bash
# Zapisać zmiany
git stash

# Z wiadomością
git stash save "Opisowa wiadomość"

# Zastosować ostatni stash
git stash apply

# Zastosować i usunąć
git stash pop

# Listować stashy
git stash list
```

---

## Rebase

### Czym jest Rebase?

**Rebase** = Ponownie zastosować commity na innej bazie

- **Liniowa historia** : Czystsza
- **Przepisywanie** : Modyfikuje historię
- **Uwaga** : Nie rebase'ować na współdzielonych gałęziach

### Prosty Rebase

```bash
# Rebase na main
git checkout feature-branche
git rebase main

# Rebase interaktywny
git rebase -i HEAD~3
```

---

## Tagi

### Czym jest Tag?

**Tag** = Wskaźnik do konkretnego commita

- **Wersja** : Oznaczać wersje
- **Release** : Punkty release
- **Referencja** : Stabilna referencja

### Utworzyć Tag

```bash
# Tag lekki
git tag v1.0.0

# Tag annotowany (zalecany)
git tag -a v1.0.0 -m "Wersja 1.0.0"

# Wypchnąć tag
git push origin v1.0.0
```

---

## Hooki

### Czym jest Hook?

**Hook** = Skrypt wykonywany przy określonych zdarzeniach

- **Automatyzacja** : Wykonywać akcje
- **Walidacja** : Sprawdzać przed commitem
- **Powiadomienia** : Powiadamiać po push

### Przykład Hooka

**`.git/hooks/pre-commit`:**

```bash
#!/bin/bash
# Uruchomić testy przed commitem
python -m pytest tests/

if [ $? -ne 0 ]; then
    echo "Testy nieudane, commit anulowany"
    exit 1
fi
```

---

## Polecenia zaawansowane

### Cherry-pick

```bash
# Zastosować konkretny commit
git cherry-pick <hash>
```

### Reflog

```bash
# Zobaczyć historię akcji
git reflog

# Odzyskać utracony commit
git checkout <hash>
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **Stash** : Tymczasowo zapisać
2. **Rebase** : Przepisać historię
3. **Tagi** : Oznaczać wersje
4. **Hooki** : Automatyzować akcje
5. **Polecenia zaawansowane** : Potężne narzędzia

## 🔗 Następny moduł

Przejdź do modułu [7. Dobre praktyki](./07-best-practices/README.md), aby poznać najlepsze praktyki.

