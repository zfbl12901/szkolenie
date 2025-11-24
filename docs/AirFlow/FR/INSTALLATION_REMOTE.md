# Installation Airflow - Accès distant

## 🎯 Objectif

Ce guide vous permet d'installer Apache Airflow sur une **machine A** (serveur) et d'y accéder depuis une **machine B** (client) via le réseau local.

## 📋 Prérequis

### Machine A (Serveur)
- Python 3.8+ installé
- Accès administrateur
- Connexion réseau active
- Port 8080 disponible (ou autre port)

### Machine B (Client)
- Navigateur web
- Connexion au même réseau local que la machine A

## 🔧 Installation sur Machine A

### Étape 1 : Installer Python et dépendances

**Windows :**
```powershell
# Vérifier Python
python --version

# Installer pip si nécessaire
python -m ensurepip --upgrade
```

**Linux :**
```bash
# Installer Python et pip
sudo apt update
sudo apt install python3 python3-pip python3-venv
```

### Étape 2 : Créer l'environnement Airflow

```bash
# Créer un répertoire pour Airflow
mkdir airflow-install
cd airflow-install

# Créer un environnement virtuel
python -m venv airflow-env

# Activer l'environnement
# Windows
airflow-env\Scripts\activate
# Linux
source airflow-env/bin/activate

# Installer Airflow
pip install apache-airflow

# Installer le provider PostgreSQL (optionnel, pour base de métadonnées)
pip install apache-airflow-providers-postgres
```

### Étape 3 : Configurer Airflow

```bash
# Initialiser la base de données
airflow db init

# Créer un utilisateur administrateur
airflow users create \
    --username admin \
    --firstname Admin \
    --lastname User \
    --role Admin \
    --email admin@example.com \
    --password admin123
```

### Étape 4 : Configurer l'accès réseau

**Modifier la configuration Airflow :**

1. Trouver le fichier `airflow.cfg` :
   - Windows : `%USERPROFILE%\airflow\airflow.cfg`
   - Linux : `~/airflow/airflow.cfg`

2. Modifier les paramètres suivants :

```ini
[webserver]
# Permettre l'accès depuis toutes les interfaces
web_server_host = 0.0.0.0
web_server_port = 8080

# Désactiver l'authentification basique (optionnel, pour développement)
auth_backend = airflow.api.auth.backend.basic_auth
```

**Ou créer un fichier de configuration personnalisé :**

```bash
# Créer un fichier airflow.cfg personnalisé
export AIRFLOW_HOME=/path/to/airflow
airflow config get-value webserver web_server_host
```

### Étape 5 : Configurer le pare-feu

**Windows (Firewall) :**

1. Ouvrir "Pare-feu Windows Defender"
2. "Paramètres avancés"
3. "Règles de trafic entrant" → "Nouvelle règle"
4. Type : Port
5. Port : 8080 (TCP)
6. Action : Autoriser la connexion
7. Nom : "Airflow Web Server"

**Linux (UFW) :**

```bash
# Autoriser le port 8080
sudo ufw allow 8080/tcp
sudo ufw reload
```

**Linux (firewalld) :**

```bash
# Autoriser le port 8080
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
```

### Étape 6 : Démarrer Airflow

**Terminal 1 - Web Server :**

```bash
# Activer l'environnement virtuel
source airflow-env/bin/activate  # Linux
# ou
airflow-env\Scripts\activate  # Windows

# Démarrer le serveur web
airflow webserver --port 8080 --host 0.0.0.0
```

**Terminal 2 - Scheduler :**

```bash
# Activer l'environnement virtuel
source airflow-env/bin/activate  # Linux
# ou
airflow-env\Scripts\activate  # Windows

# Démarrer le scheduler
airflow scheduler
```

### Étape 7 : Obtenir l'adresse IP de la Machine A

**Windows :**
```powershell
ipconfig
# Chercher "Adresse IPv4" (ex: 192.168.1.100)
```

**Linux :**
```bash
ip addr show
# ou
hostname -I
# Chercher l'adresse IP (ex: 192.168.1.100)
```

## 🌐 Accès depuis Machine B

### Étape 1 : Vérifier la connectivité

**Depuis Machine B :**

```bash
# Tester la connexion
ping 192.168.1.100  # Remplacer par l'IP de la Machine A

# Tester le port
telnet 192.168.1.100 8080
# ou
curl http://192.168.1.100:8080
```

### Étape 2 : Accéder à l'interface web

1. Ouvrir un navigateur sur Machine B
2. Aller sur : `http://192.168.1.100:8080`
   - Remplacer `192.168.1.100` par l'IP de la Machine A
3. Se connecter avec :
   - **Username** : `admin`
   - **Password** : `admin123` (ou celui que vous avez créé)

## 🔒 Sécurité

### Recommandations

1. **Changer le mot de passe par défaut**
   ```bash
   airflow users set-password admin
   ```

2. **Utiliser HTTPS** (en production)
   - Configurer un reverse proxy (nginx, Apache)
   - Utiliser des certificats SSL

3. **Limiter l'accès réseau**
   - Utiliser un VPN
   - Restreindre les IPs autorisées dans le firewall

4. **Authentification renforcée**
   - Utiliser OAuth
   - Intégrer avec LDAP/Active Directory

### Configuration sécurisée

**Modifier `airflow.cfg` :**

```ini
[webserver]
# Activer l'authentification
auth_backend = airflow.api.auth.backend.basic_auth

# Limiter les hôtes autorisés (optionnel)
hostname_callable = airflow.utils.net.get_hostname
```

## 🐛 Dépannage

### Problème : Impossible de se connecter depuis Machine B

**Solutions :**

1. **Vérifier le pare-feu**
   ```bash
   # Windows
   netsh advfirewall firewall show rule name="Airflow Web Server"
   
   # Linux
   sudo ufw status
   ```

2. **Vérifier que Airflow écoute sur 0.0.0.0**
   ```bash
   # Vérifier les ports ouverts
   netstat -an | grep 8080
   # Doit afficher : 0.0.0.0:8080
   ```

3. **Vérifier la configuration réseau**
   - Les deux machines sont sur le même réseau
   - Pas de VPN qui bloque la connexion
   - Pas de proxy qui interfère

### Problème : Erreur "Connection refused"

**Solutions :**

1. Vérifier que le serveur web est démarré
2. Vérifier le port (8080 par défaut)
3. Vérifier les logs Airflow :
   ```bash
   # Logs du webserver
   tail -f ~/airflow/logs/webserver.log
   ```

### Problème : Erreur d'authentification

**Solutions :**

1. Vérifier les credentials
2. Recréer l'utilisateur si nécessaire :
   ```bash
   airflow users create \
       --username admin \
       --role Admin \
       --email admin@example.com \
       --password nouveau_mot_de_passe
   ```

## 📝 Configuration avancée

### Utiliser un reverse proxy (nginx)

**Installation nginx :**

```bash
# Linux
sudo apt install nginx

# Configuration nginx
sudo nano /etc/nginx/sites-available/airflow
```

**Configuration nginx :**

```nginx
server {
    listen 80;
    server_name airflow.local;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

**Activer la configuration :**

```bash
sudo ln -s /etc/nginx/sites-available/airflow /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### Service systemd (Linux)

**Créer un service pour Airflow :**

```bash
sudo nano /etc/systemd/system/airflow-webserver.service
```

**Contenu :**

```ini
[Unit]
Description=Airflow webserver daemon
After=network.target

[Service]
User=airflow
Group=airflow
Type=simple
ExecStart=/path/to/airflow-env/bin/airflow webserver
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

**Activer le service :**

```bash
sudo systemctl daemon-reload
sudo systemctl enable airflow-webserver
sudo systemctl start airflow-webserver
```

## 📊 Vérification

### Test de connexion

**Depuis Machine B :**

```bash
# Test HTTP
curl http://192.168.1.100:8080/health

# Test avec authentification
curl -u admin:admin123 http://192.168.1.100:8080/api/v1/dags
```

### Vérifier les logs

**Sur Machine A :**

```bash
# Logs du webserver
tail -f ~/airflow/logs/webserver.log

# Logs du scheduler
tail -f ~/airflow/logs/scheduler/*.log
```

## 🔗 Ressources

- [Documentation Airflow](https://airflow.apache.org/docs/)
- [Configuration Airflow](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html)
- [Sécurité Airflow](https://airflow.apache.org/docs/apache-airflow/stable/security/index.html)

---

**Note :** Cette configuration est pour un environnement de développement. Pour la production, utilisez des pratiques de sécurité renforcées (HTTPS, authentification OAuth, etc.).

