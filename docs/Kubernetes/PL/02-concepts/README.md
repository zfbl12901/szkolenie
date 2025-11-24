# 2. Podstawowe koncepcje Kubernetes

## 🎯 Cele

- Zrozumieć architekturę Kubernetes
- Opanować Pody, Nody, Klastry
- Zrozumieć Kontrolery i ReplicaSety
- Używać Namespace'ów

## 📋 Spis treści

1. [Architektura](#architektura)
2. [Pody](#pody)
3. [Nody](#nody)
4. [Kontrolery](#kontrolery)
5. [Namespace'y](#namespacey)

---

## Architektura

### Główne komponenty

**Master Node:**
- **API Server** : Punkt wejścia
- **etcd** : Baza danych
- **Scheduler** : Planuje Pody
- **Controller Manager** : Zarządza kontrolerami

**Worker Node:**
- **kubelet** : Agent na każdym nodzie
- **kube-proxy** : Sieć
- **Container Runtime** : Docker/containerd

---

## Pody

### Czym jest Pod?

**Pod** = Najmniejsza jednostka wdrożeniowa

- **Jeden lub więcej kontenerów** : Dzielą sieć/magazyn
- **Efemeryczny** : Może być odtworzony
- **Unikalne IP** : Każdy Pod ma IP

### Przykład Poda

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: app
    image: nginx
    ports:
    - containerPort: 80
```

---

## Nody

### Typy nodów

**Master Node:**
- Zarządza klastrem
- Planuje Pody
- Zarządza stanem

**Worker Node:**
- Uruchamia Pody
- Dostarcza zasoby

---

## Kontrolery

### Typy kontrolerów

**ReplicaSet:**
- Utrzymuje liczbę Podów
- Auto-healing

**Deployment:**
- Zarządza ReplicaSetami
- Rolling updates

**StatefulSet:**
- Dla aplikacji stateful
- Stabilna tożsamość

---

## Namespace'y

### Używać Namespace'ów

```bash
# Listować namespace'y
kubectl get namespaces

# Utworzyć namespace
kubectl create namespace my-namespace

# Używać namespace
kubectl get pods -n my-namespace
```

---

## 📊 Kluczowe punkty do zapamiętania

1. **Architektura** : Master i Worker Nodes
2. **Pody** : Najmniejsze jednostki
3. **Kontrolery** : Zarządzają Podami
4. **Namespace'y** : Izolacja logiczna

## 🔗 Następny moduł

Przejdź do modułu [3. Pody i Deploymenty](./03-pods-deployments/README.md).

