# 8. Projets Pratiques

## 🎯 Objectifs

- Créer une application complète
- Intégrer Qdrant dans un projet réel
- Appliquer les bonnes pratiques

## Projet 1 : Service de recherche sémantique

```java
@Service
public class SemanticSearchService {
    private final QdrantClient client;
    private final EmbeddingModel embeddingModel;
    
    public List<SearchResult> search(String query, int limit) {
        List<Float> queryVector = embeddingModel.encode(query);
        
        SearchPoints searchPoints = SearchPoints.newBuilder()
            .setCollectionName("documents")
            .addAllVector(queryVector)
            .setLimit(limit)
            .build();
        
        List<ScoredPoint> results = client.search(searchPoints).getResultList();
        return mapToSearchResults(results);
    }
}
```

## Projet 2 : Système de recommandation

```java
public List<Product> recommendSimilarProducts(long productId, int limit) {
    // Récupérer le vecteur du produit
    GetPoints getPoints = GetPoints.newBuilder()
        .setCollectionName("products")
        .addIds(productId)
        .build();
    
    PointStruct product = client.getPoints(getPoints).getResultList().get(0);
    
    // Rechercher des produits similaires
    SearchPoints searchPoints = SearchPoints.newBuilder()
        .setCollectionName("products")
        .addAllVector(product.getVectors().getVector().getDataList())
        .setLimit(limit)
        .build();
    
    return mapToProducts(client.search(searchPoints).getResultList());
}
```

---

**Félicitations ! Vous avez terminé la formation Qdrant pour Développeur Java ! 🎉**

