# Zaawansowane SQL/PostgreSQL - Optymalizacja Zapytań

## 📚 Przegląd

Ten kurs obejmuje zaawansowane techniki optymalizacji zapytań SQL/PostgreSQL, ze szczególnym naciskiem na wykorzystanie **Dalibo** do analizy i optymalizacji wydajności.

## 🎯 Cele edukacyjne

- Zrozumieć mechanizmy wykonywania zapytań PostgreSQL
- Opanować zaawansowane techniki optymalizacji
- Używać Dalibo do analizy i optymalizacji wydajności
- Interpretować kluczowe wskaźniki wydajności
- Stosować najlepsze praktyki w rzeczywistych przypadkach

## 📖 Struktura kursu

### 1. [Podstawy optymalizacji](./01-fondamentaux/README.md)
   - Architektura PostgreSQL i planista zapytań
   - Typy indeksów i ich wykorzystanie
   - Statystyki i ANALYZE

### 2. [Analiza planów wykonania](./02-plans-execution/README.md)
   - EXPLAIN i EXPLAIN ANALYZE
   - Interpretacja operacji (Seq Scan, Index Scan, itp.)
   - Koszty i czasy wykonania

### 3. [Dalibo - Narzędzie analityczne](./03-dalibo/README.md)
   - Instalacja i konfiguracja
   - Analiza zapytań z pg_stat_statements
   - Raporty wydajności
   - Automatyczne rekomendacje

### 4. [Wskaźniki wydajności](./04-indicateurs/README.md)
   - Kluczowe metryki do monitorowania
   - Interpretacja wskaźników Dalibo
   - Progi alarmowe i najlepsze praktyki

### 5. [Techniki optymalizacji](./05-techniques/README.md)
   - Optymalizacja złączeń
   - Optymalizacja agregacji
   - Optymalizacja podzapytań
   - Partycjonowanie i równoległość

### 6. [Przypadki praktyczne](./06-cas-pratiques/README.md)
   - Rzeczywiste scenariusze optymalizacji
   - Przed/po z metrykami
   - Rozwiązywanie typowych problemów

### 7. [Ćwiczenia](./07-exercices/README.md)
   - Ćwiczenia prowadzone
   - Problemy do rozwiązania
   - Skomentowane rozwiązania

## 🚀 Szybki start

1. **Wymagania wstępne**
   - PostgreSQL 12+ zainstalowany
   - Dostęp do bazy danych testowej
   - Rozszerzenie `pg_stat_statements` włączone

2. **Konfiguracja Dalibo**
   ```sql
   -- Włącz pg_stat_statements
   CREATE EXTENSION IF NOT EXISTS pg_stat_statements;
   ```

3. **Przejście przez kurs**
   - Zacznij od modułu 1 (Podstawy)
   - Postępuj zgodnie z kolejnością modułów dla logicznej progresji
   - Ćwicz z ćwiczeniami z modułu 7

## 📊 Zalecane narzędzia

- **Dalibo** : Analiza wydajności PostgreSQL
- **pgAdmin** : Interfejs graficzny dla PostgreSQL
- **psql** : Klient wiersza poleceń
- **EXPLAIN Visualizer** : Wizualizacja planów wykonania

## 📝 Konwencje

- Przykłady SQL są testowane na PostgreSQL 14+
- Metryki oparte są na typowych środowiskach produkcyjnych
- Czasy wykonania mogą się różnić w zależności od konfiguracji

## 🤝 Wkład

Ten kurs jest zaprojektowany tak, aby był rozwijany. Nie wahaj się proponować ulepszeń lub dodatkowych przypadków użycia.

## 📚 Dodatkowe zasoby

- [Oficjalna dokumentacja PostgreSQL](https://www.postgresql.org/docs/)
- [Dokumentacja Dalibo](https://dalibo.github.io/pg_qualstats/)
- [PostgreSQL Performance Tuning](https://wiki.postgresql.org/wiki/Performance_Optimization)

