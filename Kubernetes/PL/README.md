# Szkolenie Kubernetes dla Data Analyst

## 📚 Przegląd

To szkolenie poprowadzi Cię przez naukę **Kubernetes** jako Data Analyst. Kubernetes to platforma open-source do orkiestracji i zarządzania kontenerami na dużą skalę.

## 🎯 Cele szkoleniowe

- Zrozumieć Kubernetes i orkiestrację kontenerów
- Zainstalować Kubernetes (lokalnie)
- Tworzyć i zarządzać Podami i Deploymentami
- Konfigurować Serwisy i Ingress
- Zarządzać ConfigMapami i Secretami
- Używać Persistent Volumes
- Tworzyć praktyczne projekty do portfolio

## 💰 Wszystko jest darmowe!

To szkolenie używa tylko:
- ✅ **Minikube / Kind** : Darmowy lokalny Kubernetes
- ✅ **kubectl** : Darmowa CLI Kubernetes
- ✅ **Oficjalna dokumentacja** : Kompletne darmowe przewodniki
- ✅ **Tutoriale online** : Darmowe zasoby

**Całkowity budżet: 0 zł**

## 📖 Struktura szkolenia

### 1. [Rozpoczęcie z Kubernetes](./01-getting-started/README.md)
   - Zainstalować Kubernetes lokalnie
   - Podstawowe koncepcje
   - Pierwsze Pody
   - Podstawowe polecenia

### 2. [Podstawowe koncepcje](./02-concepts/README.md)
   - Architektura Kubernetes
   - Pody, Nody, Klastry
   - Kontrolery i ReplicaSety
   - Namespace'y

### 3. [Pody i Deploymenty](./03-pods-deployments/README.md)
   - Tworzyć Pody
   - Zarządzać Deploymentami
   - Skalowanie i Rolling Updates
   - Health Checks

### 4. [Serwisy](./04-services/README.md)
   - Typy serwisów
   - Service Discovery
   - Load Balancing
   - Ingress

### 5. [ConfigMaps i Secrety](./05-configmaps-secrets/README.md)
   - Zarządzać konfiguracją
   - Zarządzanie secretami
   - Zmienne środowiskowe
   - Dobre praktyki

### 6. [Persistent Volumes](./06-persistent-volumes/README.md)
   - Wolumeny Kubernetes
   - Persistent Volumes
   - Storage Classes
   - StatefulSety

### 7. [Dobre praktyki](./07-best-practices/README.md)
   - Bezpieczeństwo
   - Wydajność
   - Organizacja
   - Monitorowanie

### 8. [Projekty praktyczne](./08-projets/README.md)
   - Wdrożyć aplikację web
   - Pipeline danych z Kubernetes
   - Kompletny stack
   - Projekty do portfolio

## 🚀 Szybki start

### Wymagania wstępne

- **Docker** : Zainstalowany i działający
- **4 GB RAM** : Minimum zalecane
- **Miejsce na dysku** : 20 GB wolne

### Szybka instalacja

**Minikube (zalecane):**

```bash
# Zainstalować Minikube
# Windows
choco install minikube

# Linux
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# macOS
brew install minikube

# Uruchomić Minikube
minikube start

# Sprawdzić
kubectl get nodes
```

### Pierwszy Pod

```bash
# Utworzyć Pod
kubectl run nginx --image=nginx

# Zobaczyć Pody
kubectl get pods

# Opisać Pod
kubectl describe pod nginx
```

## 📊 Przypadki użycia dla Data Analyst

- **Orkiestracja** : Zarządzać wieloma kontenerami
- **Skalowanie** : Skalować automatycznie
- **Wdrażanie** : Łatwo wdrażać
- **Odporność** : Auto-healing
- **Pipeline'y danych** : Orkiestrować pipeline'y

## 📚 Darmowe zasoby

### Oficjalna dokumentacja

- **Dokumentacja Kubernetes** : https://kubernetes.io/docs/
- **Kubernetes Playground** : https://www.katacoda.com/courses/kubernetes
- **Minikube** : https://minikube.sigs.k8s.io/

