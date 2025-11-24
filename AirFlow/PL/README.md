# Szkolenie Apache Airflow dla Data Analyst

## 📚 Przegląd

Ten kurs prowadzi Cię przez naukę **Apache Airflow** jako Data Analyst. Airflow to platforma open-source do orkiestracji i automatyzacji złożonych przepływów pracy danych.

## 🎯 Cele edukacyjne

- Zrozumieć Apache Airflow i jego rolę w orkiestracji ETL
- Zainstalować i skonfigurować Airflow
- Tworzyć DAGi (Directed Acyclic Graphs)
- Używać operatorów, sensorów i hooków
- Orkiestrować złożone pipeline'y danych
- Integrować z bazami danych i usługami chmurowymi
- Tworzyć praktyczne projekty do portfolio

## 💰 Wszystko jest bezpłatne!

Ten kurs wykorzystuje wyłącznie:
- ✅ **Apache Airflow** : Open-source i bezpłatne
- ✅ **Python** : Bezpłatny język programowania
- ✅ **PostgreSQL/SQLite** : Bezpłatne bazy danych
- ✅ **Oficjalna dokumentacja** : Kompletne bezpłatne przewodniki

**Całkowity budżet: 0 zł**

## 📖 Struktura kursu

### 1. [Rozpoczęcie z Airflow](./01-getting-started/README.md)
   - Zainstalować Airflow
   - Podstawowa konfiguracja
   - Interfejs web Airflow
   - Pierwsze DAGi

### 2. [Podstawowe koncepcje](./02-concepts/README.md)
   - DAGi (Directed Acyclic Graphs)
   - Zadania i zależności
   - Harmonogramowanie i wyzwalacze
   - Zmienne i połączenia

### 3. [Operatory](./03-operators/README.md)
   - Operatory Python
   - Operatory SQL
   - Operatory Bash
   - Operatory niestandardowe

### 4. [Sensory](./04-sensors/README.md)
   - FileSensor
   - SqlSensor
   - HttpSensor
   - Sensory niestandardowe

### 5. [Hooki](./05-hooks/README.md)
   - Hooki baz danych
   - Hooki chmurowe (AWS, Azure)
   - Hooki HTTP
   - Tworzyć hooki niestandardowe

### 6. [Zmienne i Połączenia](./06-variables-connections/README.md)
   - Zarządzać zmiennymi
   - Konfigurować połączenia
   - Bezpieczeństwo i dobre praktyki
   - Zmienne dynamiczne

### 7. [Dobre praktyki](./07-best-practices/README.md)
   - Struktura DAGów
   - Obsługa błędów
   - Wydajność i optymalizacja
   - Testy i debugowanie

### 8. [Projekty praktyczne](./08-projets/README.md)
   - Kompletny pipeline ETL
   - Orkiestracja przepływów pracy
   - Integracja z bazami danych
   - Projekty do portfolio

## 🚀 Szybki start

### Wymagania wstępne

- **Python 3.8+** : Zainstalowany w systemie
- **pip** : Menedżer pakietów Python
- **PostgreSQL** (opcjonalne) : Dla bazy metadanych

### Szybka instalacja

```bash
# Utworzyć środowisko wirtualne
python -m venv airflow-env

# Aktywować środowisko
# Windows
airflow-env\Scripts\activate
# Linux/Mac
source airflow-env/bin/activate

# Zainstalować Airflow
pip install apache-airflow

# Zainicjalizować bazę danych
airflow db init

# Utworzyć użytkownika admin
airflow users create \
    --username admin \
    --firstname Admin \
    --lastname User \
    --role Admin \
    --email admin@example.com

# Uruchomić serwer web
airflow webserver --port 8080

# W innym terminalu, uruchomić scheduler
airflow scheduler
```

### Dostęp do interfejsu web

1. Otworzyć przeglądarkę
2. Przejść do : `http://localhost:8080`
3. Zalogować się z utworzonymi credentials

## 📊 Przypadki użycia dla Data Analyst

- **Orkiestracja ETL** : Koordynować pipeline'y danych
- **Harmonogramowanie** : Planować zadania cykliczne
- **Monitorowanie** : Monitorować wykonanie przepływów pracy
- **Obsługa błędów** : Automatyczne ponowienie i alerty
- **Integracja** : Łączyć wiele narzędzi i usług

## ⚠️ Instalacja zdalna

Jeśli instalujesz Airflow na maszynie A i chcesz uzyskać do niej dostęp z maszyny B, zobacz przewodnik [Instalacja i dostęp zdalny](./INSTALLATION_REMOTE.md).

## 📚 Bezpłatne zasoby

### Oficjalna dokumentacja

- **Apache Airflow** : https://airflow.apache.org/docs/
  - Kompletne przewodniki
  - Tutoriale krok po kroku
  - Przykłady kodu
  - Referencja API

- **GitHub Airflow** : https://github.com/apache/airflow
  - Kod źródłowy
  - Przykłady DAGów
  - Wkłady

### Zasoby zewnętrzne

- **YouTube** : Tutoriale Airflow
- **Medium** : Artykuły i przewodniki
- **Stack Overflow** : Pytania i odpowiedzi

## 🎓 Certyfikacje (opcjonalne)

### Apache Airflow (brak oficjalnej certyfikacji)

- **Szkolenie** : Bezpłatna dokumentacja i tutoriale
- **Czas trwania** : 2-4 tygodnie
- **Poziom** : Średniozaawansowany do zaawansowanego

## 📝 Konwencje

- Wszystkie przykłady używają Python 3.8+
- DAGi są testowane na Airflow 2.x
- Ścieżki mogą się różnić w zależności od systemu operacyjnego
- Porty domyślne mogą być modyfikowane

## 🤝 Wkład

Ten kurs jest zaprojektowany jako rozwijający się. Nie wahaj się proponować ulepszeń lub dodatkowych przypadków użycia.

## 📚 Dodatkowe zasoby

- [Dokumentacja Apache Airflow](https://airflow.apache.org/docs/)
- [GitHub Apache Airflow](https://github.com/apache/airflow)
- [Społeczność Airflow](https://airflow.apache.org/community/)
- [Przykłady Airflow](https://github.com/apache/airflow/tree/main/airflow/example_dags)

