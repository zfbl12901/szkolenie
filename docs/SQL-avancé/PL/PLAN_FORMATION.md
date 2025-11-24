# Plan Kursu Zaawansowanego SQL/PostgreSQL - Optymalizacja Zapytań

## 📋 Przegląd

Ten dokument przedstawia kompletny plan kursu dotyczącego optymalizacji SQL/PostgreSQL ze szczególnym naciskiem na Dalibo i wskaźniki wydajności.

## 🎯 Cele edukacyjne

1. **Zrozumieć** wewnętrzne mechanizmy PostgreSQL
2. **Analizować** plany wykonania i identyfikować problemy
3. **Używać** Dalibo do automatycznej analizy
4. **Interpretować** wskaźniki wydajności
5. **Stosować** zaawansowane techniki optymalizacji
6. **Rozwiązywać** rzeczywiste problemy wydajnościowe

## 📚 Struktura kursu

### Moduł 1 : Podstawy optymalizacji
**Szacowany czas :** 2-3 godziny

**Treść :**
- Architektura PostgreSQL i planista zapytań
- Typy indeksów (B-tree, Hash, GIN, GiST, BRIN)
- Statystyki i ANALYZE
- Parametry kosztów

**Nabyte umiejętności :**
- Zrozumieć, jak PostgreSQL wykonuje zapytania
- Wybrać odpowiedni typ indeksu
- Utrzymywać statystyki na bieżąco

### Moduł 2 : Analiza planów wykonania
**Szacowany czas :** 2-3 godziny

**Treść :**
- EXPLAIN i EXPLAIN ANALYZE
- Typy operacji (Seq Scan, Index Scan, Hash Join, itp.)
- Interpretacja kosztów
- Sygnały alarmowe

**Nabyte umiejętności :**
- Czytać i interpretować plany wykonania
- Identyfikować problematyczne operacje
- Rozumieć metryki wydajności

### Moduł 3 : Dalibo - Narzędzie analityczne
**Szacowany czas :** 3-4 godziny

**Treść :**
- Instalacja i konfiguracja
- pg_stat_statements
- pg_qualstats
- pg_stat_monitor
- Raporty i wizualizacje
- Automatyczne rekomendacje

**Nabyte umiejętności :**
- Instalować i konfigurować narzędzia Dalibo
- Analizować statystyki zapytań
- Automatycznie identyfikować brakujące indeksy
- Generować raporty wydajności

### Moduł 4 : Wskaźniki wydajności
**Szacowany czas :** 2-3 godziny

**Treść :**
- Metryki systemowe (CPU, pamięć, połączenia)
- Metryki zapytań (czas, częstotliwość, cache)
- Metryki indeksów (użycie, bloat)
- Metryki I/O
- Progi alarmowe
- Panele kontrolne

**Nabyte umiejętności :**
- Monitorować kluczowe metryki
- Definiować odpowiednie progi alarmowe
- Tworzyć panele kontrolne monitoringu

### Moduł 5 : Techniki optymalizacji
**Szacowany czas :** 3-4 godziny

**Treść :**
- Optymalizacja złączeń
- Optymalizacja agregacji
- Optymalizacja podzapytań
- Partycjonowanie (Range, List, Hash)
- Równoległość
- Optymalizacja typów danych

**Nabyte umiejętności :**
- Optymalizować różne typy zapytań
- Skutecznie wykorzystywać partycjonowanie
- Wykorzystywać równoległość PostgreSQL

### Moduł 6 : Przypadki praktyczne
**Szacowany czas :** 3-4 godziny

**Treść :**
- 6 rzeczywistych przypadków optymalizacji
- Analiza przed/po z metrykami
- Wykorzystanie Dalibo do analizy
- Rozwiązywanie typowych problemów

**Nabyte umiejętności :**
- Stosować techniki na rzeczywistych przypadkach
- Mierzyć wpływ optymalizacji
- Rozwiązywać złożone problemy

### Moduł 7 : Ćwiczenia
**Szacowany czas :** 4-6 godzin

**Treść :**
- 6 progresywnych ćwiczeń (Początkujący → Zaawansowany)
- Skomentowane rozwiązania
- Problemy do rozwiązania

**Nabyte umiejętności :**
- Ćwiczyć poznane techniki
- Rozwiązywać problemy samodzielnie
- Konsolidować wiedzę

## 📊 Wskaźniki Dalibo objęte kursem

### Główne narzędzia

1. **pg_stat_statements**
   - Identyfikacja wolnych zapytań
   - Analiza czasów wykonania
   - Wykrywanie wysokiego I/O
   - Współczynnik trafień cache na zapytanie

2. **pg_qualstats**
   - Statystyki dotyczące predykatów
   - Automatyczna identyfikacja brakujących indeksów
   - Rekomendacje indeksów
   - Analiza warunków WHERE/JOIN

3. **pg_stat_monitor**
   - Monitoring z agregacją czasową
   - Analiza błędów
   - Wiele planów wykonania
   - Wiadra czasowe

### Kluczowe monitorowane metryki

| Metryka | Narzędzie | Próg alarmowy |
|---------|-----------|---------------|
| Średni czas wykonania | pg_stat_statements | > 1000ms |
| Współczynnik trafień cache | pg_stat_statements | < 95% |
| Brakujące indeksy | pg_qualstats | Częstotliwość > 1000 |
| Zapytania z I/O tymczasowym | pg_stat_statements | > 0 |
| Połączenia idle in transaction | pg_stat_activity | > 5% |

## 🎓 Zalecane ścieżki uczenia się

### Pełna ścieżka (16-20 godzin)
1. Moduł 1 : Podstawy
2. Moduł 2 : Plany wykonania
3. Moduł 3 : Dalibo
4. Moduł 4 : Wskaźniki
5. Moduł 5 : Techniki
6. Moduł 6 : Przypadki praktyczne
7. Moduł 7 : Ćwiczenia

### Przyspieszona ścieżka (8-10 godzin)
1. Moduł 1 : Podstawy (szybka powtórka)
2. Moduł 2 : Plany wykonania
3. Moduł 3 : Dalibo (fokus na pg_stat_statements i pg_qualstats)
4. Moduł 4 : Wskaźniki (kluczowe metryki)
5. Moduł 6 : Przypadki praktyczne (2-3 przypadki)
6. Moduł 7 : Ćwiczenia (poziom średni)

### Ścieżka ekspercka (4-6 godzin)
1. Moduł 3 : Dalibo (pogłębienie)
2. Moduł 4 : Wskaźniki (zaawansowane panele)
3. Moduł 5 : Techniki (partycjonowanie, równoległość)
4. Moduł 7 : Ćwiczenia (poziom zaawansowany)

## 🛠️ Wymagania techniczne

### Wymagana wiedza
- Podstawy SQL (SELECT, JOIN, GROUP BY, itp.)
- Podstawowa znajomość PostgreSQL
- Dostęp do instancji PostgreSQL (12+)

### Zalecane środowisko
- PostgreSQL 12+ zainstalowany
- Dostęp superużytkownika do instalacji rozszerzeń
- Baza danych testowa z realistycznymi danymi
- Narzędzia : psql, pgAdmin (opcjonalnie)

### Wymagane rozszerzenia
```sql
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;
CREATE EXTENSION IF NOT EXISTS pg_qualstats;  -- Opcjonalne ale zalecane
CREATE EXTENSION IF NOT EXISTS pg_stat_monitor;  -- Opcjonalne
```

## 📈 Postęp i ocena

### Punkty kontrolne

1. **Po Module 2** : Zdolność do interpretacji planu wykonania
2. **Po Module 3** : Zdolność do użycia Dalibo do identyfikacji problemów
3. **Po Module 5** : Zdolność do optymalizacji różnych typów zapytań
4. **Po Module 7** : Zdolność do samodzielnego rozwiązywania złożonych problemów

### Kryteria sukcesu

- ✅ Poprawnie interpretować plan wykonania
- ✅ Identyfikować problemy wydajnościowe z Dalibo
- ✅ Tworzyć odpowiednie indeksy
- ✅ Optymalizować wolne zapytanie (poprawa > 50%)
- ✅ Konfigurować monitoring kluczowych wskaźników

## 🔗 Dodatkowe zasoby

### Oficjalna dokumentacja
- [Dokumentacja PostgreSQL](https://www.postgresql.org/docs/)
- [Dalibo GitHub](https://github.com/dalibo)
- [pg_stat_statements](https://www.postgresql.org/docs/current/pgstatstatements.html)

### Zalecane narzędzia
- **pgBadger** : Analiza logów PostgreSQL
- **pg_activity** : Monitoring w czasie rzeczywistym
- **HypoPG** : Test hipotetycznych indeksów
- **explain.dalibo.com** : Wizualizacja planów

### Społeczności
- PostgreSQL Polska
- Stack Overflow (tag: postgresql)
- Reddit r/PostgreSQL

## 📝 Uwagi pedagogiczne

### Podejście pedagogiczne
- **Teoretyczne** : Koncepcje wyjaśnione z przykładami
- **Praktyczne** : Rzeczywiste przypadki i ćwiczenia
- **Progresywne** : Od prostego do złożonego
- **Autonomiczne** : Kompletna dokumentacja do samokształcenia

### Wskazówki dla trenerów
1. Zacznij od konkretnych przykładów
2. Używaj systematycznie EXPLAIN ANALYZE
3. Pokaż wpływ przed/po optymalizacjach
4. Zachęcaj do eksperymentowania
5. Twórz powiązania między modułami

### Wskazówki dla uczniów
1. Ćwicz regularnie
2. Testuj na realistycznych danych
3. Dokumentuj swoje optymalizacje
4. Mierz wpływ systematycznie
5. Wracaj do podstaw w razie potrzeby

## 🎯 Oczekiwane rezultaty

Po ukończeniu tego kursu będziesz w stanie :

1. ✅ Analizować i optymalizować złożone zapytania SQL
2. ✅ Używać Dalibo do automatycznej identyfikacji problemów
3. ✅ Interpretować wskaźniki wydajności i definiować alerty
4. ✅ Stosować odpowiednie techniki optymalizacji
5. ✅ Rozwiązywać problemy wydajnościowe w produkcji
6. ✅ Wdrożyć skuteczny system monitoringu

---

**Ostatnia aktualizacja :** 2024
**Wersja :** 1.0

