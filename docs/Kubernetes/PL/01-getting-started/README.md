# 1. Rozpoczęcie z Kubernetes

## 🎯 Cele

- Zrozumieć Kubernetes
- Zainstalować Kubernetes lokalnie
- Zrozumieć podstawowe koncepcje
- Utworzyć pierwszy Pod

## 📋 Spis treści

1. [Wprowadzenie do Kubernetes](#wprowadzenie-do-kubernetes)
2. [Instalacja](#instalacja)
3. [Podstawowe koncepcje](#podstawowe-koncepcje)
4. [Pierwsze Pody](#pierwsze-pody)
5. [Podstawowe polecenia](#podstawowe-polecenia)

---

## Wprowadzenie do Kubernetes

### Czym jest Kubernetes?

**Kubernetes (K8s)** = Platforma orkiestracji kontenerów

- **Orkiestracja** : Zarządza wieloma kontenerami
- **Skalowanie** : Automatyczne skalowanie
- **Auto-healing** : Uruchamia ponownie nieudane kontenery
- **Load Balancing** : Rozkład ruchu
- **Rolling Updates** : Aktualizacje bez przestojów

### Dlaczego Kubernetes dla Data Analyst?

- **Orkiestracja** : Zarządzać wieloma usługami
- **Skalowanie** : Dostosować do potrzeb
- **Odporność** : Auto-healing
- **Wdrażanie** : Łatwo wdrażać

---

## Instalacja

### Minikube (zalecane)

**Windows:**
```bash
# Z Chocolatey
choco install minikube

# Uruchomić
minikube start

# Sprawdzić
kubectl get nodes
```

**Linux:**
```bash
# Zainstalować Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Uruchomić
minikube start

# Sprawdzić
kubectl get nodes
```

**macOS:**
```bash
# Z Homebrew
brew install minikube

# Uruchomić
minikube start

# Sprawdzić
kubectl get nodes
```

### Zainstalować kubectl

**kubectl** = CLI dla Kubernetes

**Windows:**
```bash
# Z Chocolatey
choco install kubernetes-cli
```

**Linux:**
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

**macOS:**
```bash
# Z Homebrew
brew install kubectl
```

---

## Podstawowe koncepcje

### Klaster

**Klaster** = Zbiór maszyn (nodów)

- **Master Node** : Zarządza klastrem
- **Worker Nodes** : Uruchamiają kontenery

### Pod

**Pod** = Najmniejsza jednostka wdrożeniowa w Kubernetes

- **Kontenery** : Jeden lub więcej kontenerów
- **Wspólne zasoby** : Sieć i magazyn
- **Efemeryczny** : Może być tworzony/usuwany

---

## Pierwsze Pody

### Utworzyć prosty Pod

```bash
# Utworzyć Pod
kubectl run nginx --image=nginx

# Zobaczyć Pody
kubectl get pods

# Opisać Pod
kubectl describe pod nginx

# Logi Poda
kubectl logs nginx
```

### Pod z YAML

**nginx-pod.yaml:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

**Utworzyć Pod:**
```bash
kubectl apply -f nginx-pod.yaml

# Zobaczyć Pod
kubectl get pods

# Usunąć Pod
kubectl delete -f nginx-pod.yaml
```

---

## Podstawowe polecenia

### Zarządzanie Podami

```bash
# Listować Pody
kubectl get pods

# Opisać Pod
kubectl describe pod pod-name

# Logi Poda
kubectl logs pod-name

# Wykonać polecenie w Podzie
kubectl exec -it pod-name -- bash

# Usunąć Pod
kubectl delete pod pod-name
```

### Informacje

```bash
# Zobaczyć nody
kubectl get nodes

# Zobaczyć namespace'y
kubectl get namespaces

# Informacje klastra
kubectl cluster-info

# Wersja
kubectl version
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **Kubernetes** orkiestruje kontenery
2. **Pody** to najmniejsze jednostki
3. **kubectl** to główna CLI
4. **Minikube/Kind** dla lokalnego Kubernetes
5. **YAML** do definiowania zasobów

## 🔗 Następny moduł

Przejdź do modułu [2. Podstawowe koncepcje](./02-concepts/README.md), aby pogłębić.

