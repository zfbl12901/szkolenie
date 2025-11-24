# 7. Dobre praktyki Git

## 🎯 Cele

- Skuteczne wiadomości commit
- Struktura projektu
- Kompletny .gitignore
- Dokumentacja
- Optymalny workflow

## 📋 Spis treści

1. [Wiadomości commit](#wiadomości-commit)
2. [Struktura projektu](#struktura-projektu)
3. [.gitignore](#gitignore)
4. [Dokumentacja](#dokumentacja)
5. [Workflow](#workflow)

---

## Wiadomości commit

### Zalecany format

```
Type: Krótki opis (max 50 znaków)

Szczegółowy opis jeśli potrzebny (72 znaki na linię)

- Punkt 1
- Punkt 2
```

### Typy commitów

- **feat** : Nowa funkcja
- **fix** : Poprawka błędu
- **docs** : Dokumentacja
- **style** : Formatowanie
- **refactor** : Refaktoryzacja
- **test** : Testy
- **chore** : Zadania konserwacyjne

---

## Struktura projektu

### Zalecana struktura

```
moj-projekt/
├── README.md
├── .gitignore
├── LICENSE
├── requirements.txt
├── src/
│   ├── __init__.py
│   └── main.py
├── tests/
│   └── test_main.py
└── docs/
    └── guide.md
```

---

## .gitignore

### Kompletny .gitignore dla Pythona

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

## Dokumentacja

### Dokumentacja kodu

**Docstringi Pythona:**

```python
def analizuj_dane(plik):
    """
    Analizuje plik danych CSV.
    
    Args:
        plik (str): Ścieżka do pliku CSV
        
    Returns:
        dict: Słownik ze statystykami
    """
    # Kod...
```

---

## Workflow

### Zalecany workflow

1. **Utworzyć gałąź** : Dla każdej funkcji
2. **Commitować regularnie** : Małe częste commity
3. **Testować** : Przed wypchnięciem
4. **Pull Request** : Do review
5. **Łączyć** : Po zatwierdzeniu

### Złote zasady

- **Jeden commit = Jedna logiczna zmiana**
- **Jasne i opisowe wiadomości**
- **Testować przed wypchnięciem**
- **Nigdy nie force push na main**
- **Synchronizować regularnie**

---

## 📊 Kluczowe punkty do zapamiętania

1. **Wiadomości** : Jasne i ustrukturyzowane
2. **Struktura** : Zorganizowana i logiczna
3. **.gitignore** : Kompletny i dostosowany
4. **Dokumentacja** : README i docstringi
5. **Workflow** : Regularny i spójny

## 🔗 Następny moduł

Przejdź do modułu [8. Projekty praktyczne](./08-projets/README.md), aby tworzyć kompletne projekty.

