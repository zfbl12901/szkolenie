# 7. Najlepsze praktyki

## 🎯 Cele

- Optymalizować wydajność
- Zarządzać wymiarami wektorów
- Wybierać odpowiednią odległość
- Strukturyzować metadane

## Wymiary wektorów

- **128-256** : Do małych zbiorów danych
- **384-512** : Do embeddings tekstu (sentence-transformers)
- **768-1536** : Do zaawansowanych modeli (BERT, etc.)

## Odległość

- **COSINE** : Zalecana do tekstów, embeddings znormalizowane
- **EUCLID** : Do danych numerycznych
- **DOT** : Do wektorów nieznormalizowanych

---

**Następny krok :** [Projekty praktyczne](./08-projets/README.md)

