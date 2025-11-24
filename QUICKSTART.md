# 🚀 Guide de Démarrage Rapide

## Étape 1 : Installation

```bash
# Installer MkDocs et le thème Material
pip install -r requirements.txt
```

## Étape 2 : Tester Localement

```bash
# Lancer le serveur de développement
mkdocs serve
```

Ouvrez votre navigateur : **http://127.0.0.1:8000**

Vous verrez votre documentation en direct ! Les modifications sont automatiquement rechargées.

## Étape 3 : Configuration GitHub Pages

### 3.1 Configurer le dépôt GitHub

1. Allez dans **Settings** > **Pages**
2. Sous **Source**, sélectionnez la branche `gh-pages`
3. Cliquez sur **Save**

### 3.2 Mettre à jour mkdocs.yml

Dans `mkdocs.yml`, modifiez ces lignes :

```yaml
site_url: https://VOTRE-USERNAME.github.io/szkolenie/
repo_url: https://github.com/VOTRE-USERNAME/szkolenie
```

Remplacez `VOTRE-USERNAME` par votre nom d'utilisateur GitHub.

## Étape 4 : Déploiement

### Option A : Déploiement Manuel

```bash
# Construire et déployer en une commande
mkdocs gh-deploy
```

Cette commande :
- Construit votre site
- Crée/met à jour la branche `gh-pages`
- Pousse les changements

### Option B : Déploiement Automatique (Recommandé)

Le fichier `.github/workflows/deploy.yml` est déjà configuré !

```bash
# Il suffit de pousser vos changements
git add .
git commit -m "Configuration MkDocs"
git push origin main
```

GitHub Actions va automatiquement :
1. ✅ Détecter le push sur `main`
2. ✅ Installer les dépendances
3. ✅ Construire le site
4. ✅ Déployer sur GitHub Pages

Votre site sera disponible à : `https://VOTRE-USERNAME.github.io/szkolenie/`

## 🎨 Personnalisation

### Changer les Couleurs

Dans `mkdocs.yml` :

```yaml
theme:
  palette:
    - scheme: default
      primary: indigo      # Changez cette couleur
      accent: indigo       # Changez cette couleur
```

Couleurs disponibles : `red`, `pink`, `purple`, `deep purple`, `indigo`, `blue`, `light blue`, `cyan`, `teal`, `green`, `light green`, `lime`, `yellow`, `amber`, `orange`, `deep orange`

### Ajouter un Logo

1. Ajoutez votre logo dans le dossier `docs/assets/`
2. Dans `mkdocs.yml` :

```yaml
theme:
  logo: assets/logo.png
  favicon: assets/favicon.ico
```

### Modifier la Page d'Accueil

Éditez le fichier `index.md` à la racine du projet.

## 📝 Ajouter du Contenu

### Structure d'un Fichier Markdown

```markdown
# Titre Principal

## Section 1

Votre contenu ici...

### Sous-section

- Point 1
- Point 2

## Section 2

### Bloc de Code

\`\`\`python
def hello():
    print("Hello, World!")
\`\`\`

### Note Importante

!!! note "Titre de la Note"
    Contenu de la note

!!! warning "Attention"
    Message d'avertissement

!!! tip "Astuce"
    Conseil utile
```

### Types d'Admonitions

- `note` : Information générale
- `abstract` : Résumé
- `info` : Information
- `tip` : Astuce
- `success` : Succès
- `question` : Question
- `warning` : Avertissement
- `failure` : Échec
- `danger` : Danger
- `bug` : Bug
- `example` : Exemple
- `quote` : Citation

## 🔍 Recherche

La recherche est automatiquement activée et supporte :
- Français
- Anglais
- Polonais

## 📊 Statistiques GitHub Pages

Après le déploiement, vous pouvez voir les statistiques dans :
**Settings** > **Pages** > **View deployment**

## ❓ Problèmes Courants

### Le site ne se construit pas

```bash
# Vérifier les erreurs de configuration
mkdocs build --verbose
```

### GitHub Actions échoue

1. Vérifiez que les permissions sont correctes dans **Settings** > **Actions** > **General**
2. Activez **Read and write permissions**

### Les changements ne s'affichent pas

1. Videz le cache de votre navigateur (Ctrl + F5)
2. Attendez 2-3 minutes pour la propagation GitHub Pages

## 🎓 Ressources

- [Documentation MkDocs](https://www.mkdocs.org/)
- [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)
- [Guide Markdown](https://www.markdownguide.org/)

---

**Besoin d'aide ?** Ouvrez une issue sur GitHub ! 🚀

