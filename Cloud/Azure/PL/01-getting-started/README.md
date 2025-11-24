# 1. Rozpoczęcie z Azure

## 🎯 Cele

- Utworzenie bezpłatnego konta Azure
- Zrozumienie bezpłatnych kredytów Azure
- Nawigacja w portalu Azure
- Konfiguracja podstawowego bezpieczeństwa (Azure AD)
- Monitorowanie kosztów

## 📋 Spis treści

1. [Utworzenie bezpłatnego konta Azure](#utworzenie-bezpłatnego-konta-azure)
2. [Zrozumienie bezpłatnych kredytów](#zrozumienie-bezpłatnych-kredytów)
3. [Nawigacja w portalu Azure](#nawigacja-w-portalu-azure)
4. [Konfiguracja Azure AD (bezpieczeństwo)](#konfiguracja-azure-ad-bezpieczeństwo)
5. [Monitorowanie kosztów](#monitorowanie-kosztów)

---

## Utworzenie bezpłatnego konta Azure

### Krok 1: Rejestracja

1. **Przejść na stronę Azure**
   - URL: https://azure.microsoft.com/pl-pl/free/
   - Kliknąć "Rozpocznij bezpłatnie"

2. **Zalogować się kontem Microsoft**
   - Użyć istniejącego konta Microsoft
   - Lub utworzyć nowe konto Microsoft

3. **Weryfikacja tożsamości**
   - Kod otrzymany SMS lub email
   - Wprowadzić kod weryfikacyjny

4. **Informacje osobiste**
   - Nazwisko
   - Imię
   - Numer telefonu
   - Kraj

5. **Weryfikacja telefoniczna**
   - Automatyczne połączenie lub SMS
   - Wprowadzić kod weryfikacyjny

6. **Metoda płatności**
   - **Ważne**: Wymagana karta kredytowa, ale **nie obciążana**
   - Azure daje Ci 200 USD kredytu na 30 dni
   - Po 30 dniach: stałe bezpłatne usługi
   - Możesz usunąć kartę później (nie zalecane)

7. **Końcowa weryfikacja tożsamości**
   - Weryfikacja SMS lub połączeniem
   - Potwierdzenie konta

### Krok 2: Potwierdzenie

- Email potwierdzający otrzymany
- Konto Azure aktywne natychmiast
- Dostęp do portalu Azure
- **200 USD kredytu** dostępne na 30 dni

**⚠️ Ważne**: Nie tworzyć wielu kont z tą samą kartą kredytową (ryzyko zawieszenia).

---

## Zrozumienie bezpłatnych kredytów

### Oferta Azure bezpłatna

Azure oferuje **3 typy** bezpłatnych usług:

#### 1. Kredyt 200 USD (30 dni)

**Co możesz zrobić:**
- Testować dowolną usługę Azure
- Tworzyć maszyny wirtualne
- Używać płatnych usług
- Eksperymentować swobodnie

**Warunki:**
- Ważne 30 dni po rejestracji
- Jeśli kredyt wyczerpany przed 30 dniami: usługi zatrzymane
- Po 30 dniach: przejście na stałe bezpłatne usługi

#### 2. Bezpłatne usługi przez 12 miesięcy

**Usługi przydatne dla Data Analyst:**

- **Azure SQL Database**: Bezpłatnie do 32 GB (12 miesięcy)
- **Azure Storage**: 5 GB (12 miesięcy)
- **Azure App Service**: 60 minut/dzień (12 miesięcy)
- **Azure Functions**: 1 milion wykonań/miesiąc (zawsze bezpłatne)

**Warunki:**
- Bezpłatne przez 12 miesięcy po rejestracji
- Limity miesięczne
- Poza limitami: normalne rozliczanie

#### 3. Zawsze bezpłatne usługi

**Usługi przydatne dla Data Analyst:**

- **Azure Functions**: 1 milion wykonań/miesiąc (zawsze bezpłatne)
- **Azure Cosmos DB**: 400 RU/s (zawsze bezpłatne)
- **Azure Active Directory**: 50 000 obiektów (zawsze bezpłatne)
- **Azure DevOps**: 5 użytkowników (zawsze bezpłatne)

**Warunki:**
- Bezpłatne w nieskończoność
- Limity miesięczne
- Poza limitami: rozliczanie poza limitem

### Sprawdzenie kredytów

1. Przejść do portalu Azure
2. "Zarządzanie kosztami + rozliczenia"
3. Zobaczyć pozostałe kredyty
4. Zobaczyć użycie według usługi

---

## Nawigacja w portalu Azure

### Interfejs główny

**Kluczowe elementy:**

1. **Pasek wyszukiwania** (u góry)
   - Szybkie wyszukiwanie usług
   - Przykład: wpisać "SQL" aby znaleźć SQL Database

2. **Menu Azure** (ikona ☰ u góry po lewej)
   - Wszystkie usługi Azure
   - Zorganizowane według kategorii
   - Konfigurowalne ulubione

3. **Powiadomienia** (u góry po prawej)
   - Alerty i powiadomienia
   - Status wdrożeń

4. **Ustawienia** (u góry po prawej)
   - Ustawienia konta
   - Motyw (jasny/ciemny)
   - Język

5. **Cloud Shell** (ikona >_ u góry)
   - Terminal w przeglądarce
   - PowerShell lub Bash
   - Bardzo przydatne do poleceń

### Niezbędne usługi dla Data Analyst

**W menu Azure, szukać:**

- **Konta magazynu**: Przechowywanie danych
- **Data Factory**: ETL w chmurze
- **Bazy danych SQL**: Bazy danych SQL
- **Synapse Analytics**: Hurtownia danych
- **Databricks**: Analiza Big Data
- **Funkcje**: Przetwarzanie serverless

### Pierwsze połączenie

1. Zalogować się: https://portal.azure.com/
2. Eksplorować pulpit nawigacyjny
3. Kliknąć "Wszystkie usługi" aby zobaczyć wszystkie usługi
4. Użyć paska wyszukiwania aby znaleźć usługę
5. Przypiąć częste usługi do pulpitu nawigacyjnego

---

## Konfiguracja Azure AD (bezpieczeństwo)

### Czym jest Azure AD?

**Azure AD** (Azure Active Directory) = Zarządzanie tożsamością i dostępem

- Zarządzanie użytkownikami
- Zarządzanie uprawnieniami
- Zabezpieczanie dostępu do usług
- Uwierzytelnianie wieloskładnikowe (MFA)

### Najlepsze praktyki bezpieczeństwa

#### 1. Włączenie uwierzytelniania wieloskładnikowego (MFA)

**Dla konta administratora:**

1. Przejść do Azure AD
2. "Użytkownicy" → Wybrać swoje konto
3. "Uwierzytelnianie wieloskładnikowe"
4. Kliknąć "Włącz"
5. Postępować zgodnie z instrukcjami

**⚠️ Ważne**: Zawsze włączać MFA dla kont administratorów.

#### 2. Utworzenie użytkowników Azure AD (zalecane)

**Do pracy w zespole:**

1. Przejść do Azure AD
2. "Użytkownicy" → "Nowy użytkownik"
3. Nazwa użytkownika: `data-analyst@twojadomena.onmicrosoft.com`
4. Hasło tymczasowe
5. Role: "Użytkownik" (domyślnie)
6. Utworzyć użytkownika

#### 3. Role Azure (RBAC)

**Role przydatne dla Data Analyst:**

- **Współautor**: Może tworzyć i zarządzać zasobami
- **Czytelnik**: Może tylko czytać
- **Współautor konta magazynu**: Dostęp do kont magazynu
- **Współautor SQL DB**: Dostęp do baz SQL

**Przypisać rolę:**

1. Przejść do zasobu (np. Konto magazynu)
2. "Kontrola dostępu (IAM)"
3. "Dodaj" → "Dodaj przypisanie roli"
4. Wybrać rolę
5. Wybrać użytkownika

### Zalecane zasady bezpieczeństwa

1. **Silne hasła**
   - Minimum 12 znaków
   - Wymagana złożoność

2. **Wygaśnięcie hasła**
   - 90 dni (zalecane)

3. **Blokada konta**
   - Po 5 nieudanych próbach

---

## Monitorowanie kosztów

### Włączenie alertów kosztów

**Krok 1: Konfiguracja alertów**

1. Przejść do "Zarządzanie kosztami + rozliczenia"
2. "Alerty kosztów"
3. "Nowy alert kosztów"
4. Próg: 5 USD (zalecane)
5. Powiadomienie email

**Wynik**: Email otrzymany, jeśli koszty przekroczą 5 USD.

### Sprawdzenie użycia kredytów

1. "Zarządzanie kosztami + rozliczenia"
2. "Kredyty Azure"
3. Zobaczyć pozostałe kredyty
4. Zobaczyć użycie według usługi
5. Zobaczyć datę wygaśnięcia (30 dni)

### Azure Cost Management

1. "Zarządzanie kosztami + rozliczenia" → "Zarządzanie kosztami"
2. Zobaczyć koszty według usługi
3. Filtrować według okresu
4. Eksportować raporty
5. Tworzyć budżety

**⚠️ Ważne**: Sprawdzać regularnie (zalecane cotygodniowo).

### Wskazówki, aby pozostać bezpłatnym

1. **Usuwanie nieużywanych zasobów**
   - Zatrzymać nieużywane maszyny wirtualne
   - Usunąć puste konta magazynu
   - Wyczyścić grupy zasobów

2. **Używanie bezpłatnych usług**
   - Priorytetyzować zawsze bezpłatne usługi
   - Używać kredytów mądrze
   - Zatrzymywać nieużywane usługi

3. **Tworzenie budżetów**
   - "Zarządzanie kosztami" → "Budżety"
   - Utworzyć budżet 5 USD
   - Automatyczne alerty

4. **Zatrzymywanie nieużywanych usług**
   - Maszyny wirtualne: zatrzymać, gdy nieużywane
   - Bazy danych: zatrzymać lub wstrzymać
   - Konta magazynu: usunąć, jeśli puste

### Grupy zasobów

**Organizowanie zasobów:**

1. Utworzyć grupę zasobów: `rg-data-analyst-training`
2. Wszystkie zasoby szkoleniowe w tej grupie
3. Ułatwia jednorazowe usunięcie
4. Ułatwia zarządzanie kosztami

**Utworzyć grupę zasobów:**

1. "Grupy zasobów" → "Dodaj"
2. Nazwa: `rg-data-analyst-training`
3. Region: Wybrać najbliższy region
4. Utworzyć

---

## 📊 Kluczowe punkty do zapamiętania

1. **Bezpłatne konto Azure**: 200 USD kredytu (30 dni) + bezpłatne usługi
2. **Bezpłatne kredyty**: 3 typy (200 USD, 12 miesięcy, zawsze bezpłatne)
3. **Bezpieczeństwo Azure AD**: Włączyć MFA, tworzyć użytkowników
4. **Monitorowanie**: Alerty kosztów niezbędne
5. **Pozostać bezpłatnym**: Usuwać nieużywane zasoby, używać grup zasobów

## 🔗 Następny moduł

Przejdź do modułu [2. Azure Storage - Przechowywanie danych](../02-storage/README.md), aby nauczyć się przechowywać dane na Azure.

