# 1. Rozpoczęcie pracy z Qdrant

## 🎯 Cele

- Zrozumieć bazy danych wektorowych
- Zainstalować Qdrant
- Zrozumieć podstawowe koncepcje
- Pierwsze operacje

## Wprowadzenie do Qdrant

**Qdrant** = Baza danych wektorowa

- **Wektory** : Reprezentacje numeryczne (embeddings)
- **Podobieństwo** : Wyszukiwanie podobieństwa
- **AI/ML** : Zoptymalizowana do sztucznej inteligencji
- **Open-source** : Darmowa i open-source

## Instalacja

### Docker (zalecane)

```bash
docker run -p 6333:6333 qdrant/qdrant
```

### Klient Python

```bash
pip install qdrant-client
```

## Pierwszy przykład

```python
from qdrant_client import QdrantClient

client = QdrantClient(host="localhost", port=6333)

client.create_collection(
    collection_name="test_collection",
    vectors_config={
        "size": 128,
        "distance": "Cosine"
    }
)
```

---

**Następny krok :** [Kolekcje i wektory](./02-collections-vectors/README.md)

