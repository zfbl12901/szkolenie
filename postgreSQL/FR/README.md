# Pages HTML — PostgreSQL Avancé

Ce répertoire contient les pages HTML pour la formation PostgreSQL Avancé.

## Structure

- `index.html` : Plan de cours principal
- `styles.css` : Feuille de style commune
- `Module 1 - Fondamentaux essentiels/module1.html` : Module 1 complet

## Utilisation

### Option 1 : Serveur HTTP local (recommandé)

Pour que les pages fonctionnent correctement (notamment le chargement du fichier Markdown), vous devez utiliser un serveur HTTP local.

#### Avec Python 3 :
```bash
cd postgreSQL/FR
python -m http.server 8000
```

Puis ouvrez dans votre navigateur : `http://localhost:8000/index.html`

#### Avec Node.js (http-server) :
```bash
npm install -g http-server
cd postgreSQL/FR
http-server -p 8000
```

#### Avec PHP :
```bash
cd postgreSQL/FR
php -S localhost:8000
```

### Option 2 : Ouvrir directement

Vous pouvez ouvrir `index.html` directement dans votre navigateur, mais le Module 1 ne chargera pas automatiquement le contenu Markdown (problème CORS avec `file://`).

## Navigation

Le header de navigation permet de naviguer entre :
- **Plan de cours** : Vue d'ensemble de la formation
- **Module 1** : Module complet d'optimisation des requêtes

## Fonctionnalités

- ✅ Header de navigation responsive
- ✅ Conversion Markdown → HTML automatique
- ✅ Coloration syntaxique pour le code SQL
- ✅ Design moderne et responsive
- ✅ Liens internes avec défilement fluide

## Notes

- Les pages utilisent `marked.js` pour la conversion Markdown
- La coloration syntaxique utilise `highlight.js`
- Le CSS est responsive et s'adapte aux mobiles

