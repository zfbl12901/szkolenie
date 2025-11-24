# Szkolenie Docker dla Data Analyst

## 📚 Przegląd

To szkolenie poprowadzi Cię przez naukę **Docker** jako Data Analyst. Docker to platforma konteneryzacji, która pozwala tworzyć, wdrażać i uruchamiać aplikacje w izolowanych kontenerach.

## 🎯 Cele szkoleniowe

- Zrozumieć Docker i konteneryzację
- Zainstalować Docker
- Tworzyć i zarządzać kontenerami
- Budować obrazy Docker
- Używać Docker Compose
- Integrować Docker w przepływy danych
- Tworzyć praktyczne projekty do portfolio

## 💰 Wszystko jest darmowe!

To szkolenie używa tylko:
- ✅ **Docker Desktop** : Darmowy do użytku osobistego/edukacyjnego
- ✅ **Docker Hub** : Darmowy rejestr publiczny
- ✅ **Oficjalna dokumentacja** : Kompletne darmowe przewodniki
- ✅ **Tutoriale online** : Darmowe zasoby

**Całkowity budżet: 0 zł**

## 📖 Struktura szkolenia

### 1. [Rozpoczęcie z Docker](./01-getting-started/README.md)
   - Zainstalować Docker
   - Podstawowe koncepcje
   - Pierwsze kontenery
   - Podstawowe polecenia

### 2. [Kontenery](./02-containers/README.md)
   - Tworzyć kontenery
   - Zarządzać cyklem życia
   - Wykonywać polecenia
   - Logi i debugowanie

### 3. [Obrazy Docker](./03-images/README.md)
   - Zrozumieć obrazy
   - Pobierać obrazy
   - Tworzyć niestandardowe obrazy
   - Zarządzać obrazami

### 4. [Dockerfile](./04-dockerfile/README.md)
   - Pisać Dockerfile
   - Dobre praktyki
   - Optymalizować obrazy
   - Multi-stage builds

### 5. [Docker Compose](./05-docker-compose/README.md)
   - Orkiestrować wiele kontenerów
   - Plik docker-compose.yml
   - Usługi i sieci
   - Zmienne środowiskowe

### 6. [Wolumeny i Sieci](./06-volumes-networks/README.md)
   - Zarządzać wolumenami
   - Tworzyć sieci
   - Dzielić dane
   - Trwałość danych

### 7. [Dobre praktyki](./07-best-practices/README.md)
   - Bezpieczeństwo
   - Wydajność
   - Organizacja
   - Konserwacja

### 8. [Projekty praktyczne](./08-projets/README.md)
   - Konteneryzować aplikację Python
   - Pipeline danych z Docker
   - Kompletny stack z Docker Compose
   - Projekty do portfolio

## 🚀 Szybki start

### Wymagania wstępne

- **System operacyjny** : Windows, Linux lub macOS
- **4 GB RAM** : Minimum zalecane
- **Miejsce na dysku** : 20 GB wolne

### Szybka instalacja

**Windows/Mac:**
1. Pobrać Docker Desktop: https://www.docker.com/products/docker-desktop
2. Zainstalować i uruchomić Docker Desktop
3. Sprawdzić instalację: `docker --version`

**Linux:**
```bash
# Zainstalować Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Uruchomić Docker
sudo systemctl start docker
sudo systemctl enable docker

# Sprawdzić
docker --version
```

### Pierwszy kontener

```bash
# Uruchomić kontener Hello World
docker run hello-world

# Uruchomić kontener interaktywny
docker run -it ubuntu bash
```

## 📊 Przypadki użycia dla Data Analyst

- **Reprodukowalne środowiska** : To samo środowisko wszędzie
- **Izolacja** : Oddzielić zależności
- **Wdrażanie** : Łatwo wdrażać aplikacje
- **CI/CD** : Integrować w pipeline'y
- **Data Science** : Izolowane środowiska Python/R

## 📚 Darmowe zasoby

### Oficjalna dokumentacja

- **Dokumentacja Docker** : https://docs.docker.com/
- **Docker Hub** : https://hub.docker.com/
- **Docker Playground** : https://labs.play-with-docker.com/

## 🎓 Certyfikacje (opcjonalne)

### Docker Certified Associate (DCA)

- **Koszt** : ~$195
- **Przygotowanie** : Darmowa dokumentacja
- **Czas trwania** : 2-4 tygodnie
- **Poziom** : Średnio zaawansowany

