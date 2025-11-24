# 📚 Documentation de Formation - szkolenie

Documentation complète pour les formations en technologies DevOps, Data Engineering et Cloud.

## 🌐 Langues Disponibles

- 🇫🇷 Français
- 🇬🇧 English  
- 🇵🇱 Polski

## 📖 Technologies Couvertes

### DevOps & Infrastructure
- Docker
- Kubernetes
- AirFlow
- Git

### Bases de Données
- MongoDB
- ClickHouse
- Qdrant
- SQL Avancé (PostgreSQL)

### Cloud
- AWS
- Azure

## 🚀 Installation et Utilisation

### Prérequis

- Python 3.8 ou supérieur
- pip

### Installation

```bash
# Cloner le dépôt
git clone https://github.com/votre-username/szkolenie.git
cd szkolenie

# Installer les dépendances
pip install -r requirements.txt
```

### Lancer le serveur de développement local

```bash
mkdocs serve
```

Ouvrez votre navigateur à l'adresse : `http://127.0.0.1:8000`

### Construire la documentation

```bash
mkdocs build
```

Les fichiers statiques seront générés dans le dossier `site/`.

### Déployer sur GitHub Pages

```bash
mkdocs gh-deploy
```

Ou simplement pusher sur la branche `main` - le déploiement automatique est configuré via GitHub Actions !

## 📁 Structure du Projet

```
szkolenie/
├── AirFlow/          # Formation Apache AirFlow
├── Clickhouse/       # Formation ClickHouse (Data & Dev)
├── Cloud/            # Formations Cloud (AWS & Azure)
├── Docker/           # Formation Docker
├── Git/              # Formation Git
├── Kubernetes/       # Formation Kubernetes
├── MongoDB/          # Formation MongoDB
├── Qdrant/           # Formation Qdrant (Data & Dev)
├── SQL-avancé/       # Formation SQL avancé
├── mkdocs.yml        # Configuration MkDocs
├── index.md          # Page d'accueil
└── requirements.txt  # Dépendances Python
```

## 🎯 Fonctionnalités

- ✅ Navigation claire et intuitive
- ✅ Recherche intégrée multilingue
- ✅ Mode sombre/clair
- ✅ Responsive design
- ✅ Déploiement automatique sur GitHub Pages
- ✅ Support de la coloration syntaxique
- ✅ Table des matières automatique

## 🔧 Configuration

Le fichier `mkdocs.yml` contient toute la configuration :

- **Theme** : Material Design
- **Plugins** : Recherche multilingue, tags
- **Extensions** : Support complet Markdown avec PyMdown

## 📝 Contribuer

1. Forkez le projet
2. Créez une branche pour votre fonctionnalité (`git checkout -b feature/AmazingFeature`)
3. Committez vos changements (`git commit -m 'Add some AmazingFeature'`)
4. Poussez vers la branche (`git push origin feature/AmazingFeature`)
5. Ouvrez une Pull Request

## 📄 License

Ce projet est sous licence MIT.

## 🙏 Remerciements

- [MkDocs](https://www.mkdocs.org/) - Générateur de documentation
- [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) - Thème Material Design
- Tous les contributeurs !

## 📞 Contact

Pour toute question ou suggestion, n'hésitez pas à ouvrir une issue !

---

**Bonne formation ! 🚀**

