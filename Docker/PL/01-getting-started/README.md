# 1. Rozpoczęcie z Docker

## 🎯 Cele

- Zrozumieć Docker i konteneryzację
- Zainstalować Docker
- Zrozumieć podstawowe koncepcje
- Uruchomić pierwszy kontener

## 📋 Spis treści

1. [Wprowadzenie do Docker](#wprowadzenie-do-docker)
2. [Instalacja](#instalacja)
3. [Podstawowe koncepcje](#podstawowe-koncepcje)
4. [Pierwsze kontenery](#pierwsze-kontenery)
5. [Podstawowe polecenia](#podstawowe-polecenia)

---

## Wprowadzenie do Docker

### Czym jest Docker?

**Docker** = Platforma konteneryzacji

- **Kontenery** : Izolowane i lekkie środowiska
- **Przenośny** : Działa wszędzie (Windows, Linux, macOS)
- **Wydajny** : Używa mniej zasobów niż VMs
- **Szybki** : Uruchamia się w sekundach

### Dlaczego Docker dla Data Analyst?

- **Reprodukowalność** : To samo środowisko wszędzie
- **Izolacja** : Oddzielić zależności Python/R
- **Prostota** : Łatwo dzielić i wdrażać
- **Wydajność** : Szybszy niż VMs

---

## Instalacja

### Windows

1. Przejść do: https://www.docker.com/products/docker-desktop
2. Pobrać Docker Desktop dla Windows
3. Zainstalować plik `.exe`
4. Uruchomić ponownie jeśli potrzebne

### Linux

```bash
# Zaktualizować pakiety
sudo apt update

# Zainstalować zależności
sudo apt install apt-transport-https ca-certificates curl gnupg lsb-release

# Dodać klucz GPG Docker
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Dodać repozytorium Docker
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Zainstalować Docker
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io

# Uruchomić Docker
sudo systemctl start docker
sudo systemctl enable docker

# Sprawdzić
docker --version
```

### macOS

1. Przejść do: https://www.docker.com/products/docker-desktop
2. Pobrać Docker Desktop dla Mac
3. Zainstalować plik `.dmg`
4. Otworzyć Docker z Aplikacji

---

## Podstawowe koncepcje

### Obrazy

**Obraz** = Szablon tylko do odczytu do tworzenia kontenerów

- **Szablon** : Zawiera OS, aplikacje, zależności
- **Niezmienny** : Nie zmienia się po utworzeniu
- **Lekki** : Dzieli wspólne warstwy

### Kontenery

**Kontener** = Wykonywalna instancja obrazu

- **Izolowany** : Oddzielne środowisko
- **Efemeryczny** : Może być łatwo tworzony/usuwany
- **Przenośny** : Działa wszędzie gdzie Docker jest zainstalowany

---

## Pierwsze kontenery

### Hello World

```bash
# Uruchomić kontener Hello World
docker run hello-world
```

### Kontener interaktywny

```bash
# Uruchomić interaktywny kontener Ubuntu
docker run -it ubuntu bash

# W kontenerze
ls
pwd
exit
```

### Kontener w tle

```bash
# Uruchomić kontener w tle
docker run -d --name my-container nginx

# Zobaczyć uruchomione kontenery
docker ps

# Zobaczyć logi
docker logs my-container

# Zatrzymać kontener
docker stop my-container
```

---

## Podstawowe polecenia

### Zarządzanie kontenerami

```bash
# Listować uruchomione kontenery
docker ps

# Listować wszystkie kontenery
docker ps -a

# Uruchomić kontener
docker start my-container

# Zatrzymać kontener
docker stop my-container

# Usunąć kontener
docker rm my-container
```

### Zarządzanie obrazami

```bash
# Listować obrazy
docker images

# Pobrać obraz
docker pull ubuntu

# Usunąć obraz
docker rmi ubuntu
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **Docker = Konteneryzacja** do izolacji aplikacji
2. **Obrazy** to szablony, **Kontenery** to instancje
3. **Docker Hub** do znajdowania obrazów
4. **Podstawowe polecenia** : run, ps, stop, rm
5. **Przenośny** : Działa wszędzie

## 🔗 Następny moduł

Przejdź do modułu [2. Kontenery](./02-containers/README.md), aby pogłębić zarządzanie kontenerami.

