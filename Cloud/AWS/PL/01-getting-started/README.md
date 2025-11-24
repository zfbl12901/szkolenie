# 1. Rozpoczęcie z AWS

## 🎯 Cele

- Utworzenie bezpłatnego konta AWS
- Zrozumienie AWS Free Tier
- Nawigacja w konsoli AWS
- Konfiguracja podstawowego bezpieczeństwa (IAM)
- Monitorowanie kosztów

## 📋 Spis treści

1. [Utworzenie bezpłatnego konta AWS](#utworzenie-bezpłatnego-konta-aws)
2. [Zrozumienie Free Tier](#zrozumienie-free-tier)
3. [Nawigacja w konsoli AWS](#nawigacja-w-konsoli-aws)
4. [Konfiguracja IAM (bezpieczeństwo)](#konfiguracja-iam-bezpieczeństwo)
5. [Monitorowanie kosztów](#monitorowanie-kosztów)

---

## Utworzenie bezpłatnego konta AWS

### Krok 1: Rejestracja

1. **Przejść na stronę AWS**
   - URL: https://aws.amazon.com/pl/free/
   - Kliknąć "Utwórz bezpłatne konto"

2. **Wypełnić formularz**
   - Email
   - Silne hasło
   - Nazwa konta AWS

3. **Informacje kontaktowe**
   - Imię i nazwisko
   - Numer telefonu
   - Kraj

4. **Weryfikacja**
   - Kod otrzymany SMS
   - Wprowadzić kod weryfikacyjny

5. **Metoda płatności**
   - **Ważne**: Wymagana karta kredytowa, ale **nie obciążana**
   - AWS nie obciąży Cię, dopóki pozostajesz w limitach Free Tier
   - Możesz usunąć kartę później (nie zalecane)

6. **Weryfikacja tożsamości**
   - Automatyczne połączenie
   - Wprowadzić 4-cyfrowy kod

7. **Plan wsparcia**
   - Wybrać "Plan podstawowy" (bezpłatny)
   - Inne plany są płatne

### Krok 2: Potwierdzenie

- Email potwierdzający otrzymany
- Konto AWS aktywne natychmiast
- Dostęp do konsoli AWS

**⚠️ Ważne**: Nie tworzyć wielu kont z tą samą kartą kredytową (ryzyko zawieszenia).

---

## Zrozumienie Free Tier

### Typy Free Tier

AWS oferuje **3 typy** bezpłatnych usług:

#### 1. Bezpłatne usługi przez 12 miesięcy

**Usługi przydatne dla Data Analyst:**

- **Amazon EC2**: 750 godzin/miesiąc (t2.micro)
- **Amazon RDS**: 750 godzin/miesiąc
- **Amazon Redshift**: 750 godzin/miesiąc (tylko 2 miesiące)
- **Amazon Elasticsearch**: 750 godzin/miesiąc

**Warunki:**
- Bezpłatne przez 12 miesięcy po rejestracji
- Limity miesięczne
- Poza limitami: normalne rozliczanie

#### 2. Zawsze bezpłatne usługi (z limitami)

**Usługi przydatne dla Data Analyst:**

- **Amazon S3**: 5 GB przechowywania (zawsze bezpłatne)
- **AWS Lambda**: 1 milion żądań/miesiąc (zawsze bezpłatne)
- **AWS Glue**: 10 000 obiektów/miesiąc (zawsze bezpłatne)
- **Amazon Athena**: 10 GB przeskanowanych danych/miesiąc (zawsze bezpłatne)
- **Amazon CloudWatch**: 10 niestandardowych metryk (zawsze bezpłatne)

**Warunki:**
- Bezpłatne w nieskończoność
- Limity miesięczne
- Poza limitami: rozliczanie poza limitem

#### 3. Krótkoterminowe bezpłatne próby

- **Amazon Redshift**: 2 miesiące bezpłatnie
- **Amazon QuickSight**: 1 bezpłatny użytkownik

### Sprawdzenie Free Tier

1. Przejść do konsoli AWS
2. Menu "Usługi" → "Rozliczenia"
3. Kliknąć "Free Tier"
4. Zobaczyć użycie według usługi

---

## Nawigacja w konsoli AWS

### Interfejs główny

**Kluczowe elementy:**

1. **Pasek wyszukiwania** (u góry)
   - Szybkie wyszukiwanie usług
   - Przykład: wpisać "S3" aby uzyskać dostęp do Amazon S3

2. **Menu Usługi** (u góry po lewej)
   - Wszystkie usługi AWS
   - Zorganizowane według kategorii

3. **Region** (u góry po prawej)
   - Wybrać region AWS
   - **Zalecenie**: Wybrać najbliższy region
   - Przykład: `eu-west-3` (Paryż) dla Francji

4. **Nazwa konta** (u góry po prawej)
   - Ustawienia konta
   - Rozliczenia
   - Wsparcie

### Niezbędne usługi dla Data Analyst

**W menu Usługi, szukać:**

- **S3**: Przechowywanie danych
- **Glue**: ETL serverless
- **Redshift**: Hurtownia danych
- **Athena**: Zapytania SQL na S3
- **Lambda**: Przetwarzanie serverless
- **IAM**: Zarządzanie dostępem

### Pierwsze połączenie

1. Zalogować się: https://console.aws.amazon.com/
2. Eksplorować pulpit nawigacyjny
3. Kliknąć "Usługi" aby zobaczyć wszystkie usługi
4. Użyć paska wyszukiwania aby znaleźć usługę

---

## Konfiguracja IAM (bezpieczeństwo)

### Czym jest IAM?

**IAM** (Identity and Access Management) = Zarządzanie dostępem i tożsamością

- Tworzenie użytkowników
- Zarządzanie uprawnieniami
- Zabezpieczanie dostępu do usług

### Najlepsze praktyki bezpieczeństwa

#### 1. Włączenie uwierzytelniania dwuskładnikowego (MFA)

**Dla konta root:**

1. Przejść do IAM
2. Kliknąć "Aktywuj MFA"
3. Wybrać urządzenie (telefon)
4. Zeskanować kod QR aplikacją MFA
5. Wprowadzić kody weryfikacyjne

**⚠️ Ważne**: Zawsze włączać MFA dla konta root.

#### 2. Utworzenie użytkownika IAM (zalecane)

**Nie używać konta root do codziennej pracy.**

1. Przejść do IAM
2. Kliknąć "Użytkownicy" → "Dodaj użytkowników"
3. Nazwa użytkownika: `data-analyst`
4. Typ dostępu: "Dostęp programistyczny" + "Dostęp do konsoli zarządzania AWS"
5. Uprawnienia: "Dołącz istniejące zasady bezpośrednio"
   - Wybrać: `PowerUserAccess` (na początek)
   - Lub utworzyć niestandardowe uprawnienia
6. Utworzyć użytkownika
7. **Zapisać dane dostępowe** (klucz dostępu + sekret)

#### 3. Grupy IAM (opcjonalne)

Tworzenie grup do organizowania użytkowników:

1. IAM → "Grupy" → "Utwórz grupę"
2. Nazwa: `DataAnalystGroup`
3. Dołączyć zasady
4. Dodać użytkowników do grupy

### Zalecane zasady IAM dla Data Analyst

**Niezbędne zasady:**

- `AmazonS3FullAccess`: Pełny dostęp do S3
- `AWSGlueServiceRole`: Dostęp do Glue
- `AmazonRedshiftFullAccess`: Dostęp do Redshift
- `AmazonAthenaFullAccess`: Dostęp do Athena
- `AWSLambdaFullAccess`: Dostęp do Lambda

**⚠️ Zasada najmniejszych uprawnień**: Dawać tylko niezbędne uprawnienia.

---

## Monitorowanie kosztów

### Włączenie alertów rozliczeniowych

**Krok 1: Włączenie alertów**

1. Przejść do "Rozliczenia" → "Preferencje"
2. Włączyć "Otrzymuj alerty rozliczeniowe"
3. Włączyć "Otrzymuj alerty użycia Free Tier"

**Krok 2: Utworzenie alertu CloudWatch**

1. Przejść do CloudWatch
2. "Alarmy" → "Utwórz alarm"
3. Metryka: "EstimatedCharges"
4. Próg: 5 USD (zalecane)
5. Powiadomienie: Email

**Wynik**: Email otrzymany, jeśli koszty przekroczą 5 USD.

### Sprawdzenie użycia Free Tier

1. "Rozliczenia" → "Free Tier"
2. Zobaczyć użycie według usługi
3. Sprawdzić pozostałe limity
4. Monitorować daty wygaśnięcia (12 miesięcy)

### AWS Cost Explorer

1. "Rozliczenia" → "Cost Explorer"
2. Zobaczyć koszty według usługi
3. Filtrować według okresu
4. Eksportować raporty

**⚠️ Ważne**: Sprawdzać regularnie (zalecane cotygodniowo).

### Wskazówki, aby pozostać bezpłatnym

1. **Usuwanie nieużywanych zasobów**
   - Zatrzymać nieużywane instancje EC2
   - Usunąć puste buckety S3
   - Wyczyścić migawki

2. **Przestrzeganie limitów Free Tier**
   - Uważnie czytać warunki
   - Monitorować użycie
   - Ustawiać alerty

3. **Używanie bezpłatnych regionów**
   - Niektóre regiony oferują więcej bezpłatnych usług
   - Sprawdzić dostępność

4. **Zatrzymywanie nieużywanych usług**
   - Redshift: zatrzymać klaster, gdy nieużywany
   - EC2: zatrzymać instancje
   - RDS: zatrzymać bazy danych

---

## 📊 Kluczowe punkty do zapamiętania

1. **Bezpłatne konto AWS**: 200 USD kredytu + Free Tier
2. **Free Tier**: 3 typy (12 miesięcy, zawsze bezpłatne, próby)
3. **Bezpieczeństwo IAM**: Włączyć MFA, tworzyć użytkowników
4. **Monitorowanie**: Alerty rozliczeniowe niezbędne
5. **Pozostać bezpłatnym**: Usuwać nieużywane zasoby

## 🔗 Następny moduł

Przejdź do modułu [2. Amazon S3 - Przechowywanie danych](../02-s3/README.md), aby nauczyć się przechowywać dane na AWS.

