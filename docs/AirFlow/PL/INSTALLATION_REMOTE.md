# Instalacja Airflow - Dostęp zdalny

## 🎯 Cel

Ten przewodnik pozwala zainstalować Apache Airflow na **maszynie A** (serwer) i uzyskać do niej dostęp z **maszyny B** (klient) przez sieć lokalną.

## 📋 Wymagania wstępne

### Maszyna A (Serwer)
- Python 3.8+ zainstalowany
- Dostęp administratora
- Aktywne połączenie sieciowe
- Port 8080 dostępny (lub inny port)

### Maszyna B (Klient)
- Przeglądarka web
- Połączenie z tą samą siecią lokalną co maszyna A

## 🔧 Instalacja na maszynie A

### Krok 1 : Zainstalować Python i zależności

**Windows :**
```powershell
# Sprawdzić Python
python --version

# Zainstalować pip jeśli potrzeba
python -m ensurepip --upgrade
```

**Linux :**
```bash
# Zainstalować Python i pip
sudo apt update
sudo apt install python3 python3-pip python3-venv
```

### Krok 2 : Utworzyć środowisko Airflow

```bash
# Utworzyć katalog dla Airflow
mkdir airflow-install
cd airflow-install

# Utworzyć środowisko wirtualne
python -m venv airflow-env

# Aktywować środowisko
# Windows
airflow-env\Scripts\activate
# Linux
source airflow-env/bin/activate

# Zainstalować Airflow
pip install apache-airflow

# Zainstalować provider PostgreSQL (opcjonalne, dla bazy metadanych)
pip install apache-airflow-providers-postgres
```

### Krok 3 : Skonfigurować Airflow

```bash
# Zainicjalizować bazę danych
airflow db init

# Utworzyć użytkownika administratora
airflow users create \
    --username admin \
    --firstname Admin \
    --lastname User \
    --role Admin \
    --email admin@example.com \
    --password admin123
```

### Krok 4 : Skonfigurować dostęp sieciowy

**Zmodyfikować konfigurację Airflow :**

1. Znaleźć plik `airflow.cfg` :
   - Windows : `%USERPROFILE%\airflow\airflow.cfg`
   - Linux : `~/airflow/airflow.cfg`

2. Zmodyfikować następujące parametry :

```ini
[webserver]
# Zezwolić dostęp ze wszystkich interfejsów
web_server_host = 0.0.0.0
web_server_port = 8080

# Wyłączyć uwierzytelnianie podstawowe (opcjonalne, dla rozwoju)
auth_backend = airflow.api.auth.backend.basic_auth
```

**Lub utworzyć plik konfiguracyjny niestandardowy :**

```bash
# Utworzyć niestandardowy plik airflow.cfg
export AIRFLOW_HOME=/path/to/airflow
airflow config get-value webserver web_server_host
```

### Krok 5 : Skonfigurować firewall

**Windows (Firewall) :**

1. Otworzyć "Zapora Windows Defender"
2. "Ustawienia zaawansowane"
3. "Reguły ruchu przychodzącego" → "Nowa reguła"
4. Typ : Port
5. Port : 8080 (TCP)
6. Akcja : Zezwolić na połączenie
7. Nazwa : "Airflow Web Server"

**Linux (UFW) :**

```bash
# Zezwolić port 8080
sudo ufw allow 8080/tcp
sudo ufw reload
```

**Linux (firewalld) :**

```bash
# Zezwolić port 8080
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
```

### Krok 6 : Uruchomić Airflow

**Terminal 1 - Web Server :**

```bash
# Aktywować środowisko wirtualne
source airflow-env/bin/activate  # Linux
# lub
airflow-env\Scripts\activate  # Windows

# Uruchomić serwer web
airflow webserver --port 8080 --host 0.0.0.0
```

**Terminal 2 - Scheduler :**

```bash
# Aktywować środowisko wirtualne
source airflow-env/bin/activate  # Linux
# lub
airflow-env\Scripts\activate  # Windows

# Uruchomić scheduler
airflow scheduler
```

### Krok 7 : Uzyskać adres IP maszyny A

**Windows :**
```powershell
ipconfig
# Szukać "Adres IPv4" (np. 192.168.1.100)
```

**Linux :**
```bash
ip addr show
# lub
hostname -I
# Szukać adresu IP (np. 192.168.1.100)
```

## 🌐 Dostęp z maszyny B

### Krok 1 : Sprawdzić łączność

**Z maszyny B :**

```bash
# Testować połączenie
ping 192.168.1.100  # Zastąpić IP maszyny A

# Testować port
telnet 192.168.1.100 8080
# lub
curl http://192.168.1.100:8080
```

### Krok 2 : Dostęp do interfejsu web

1. Otworzyć przeglądarkę na maszynie B
2. Przejść do : `http://192.168.1.100:8080`
   - Zastąpić `192.168.1.100` IP maszyny A
3. Zalogować się z :
   - **Username** : `admin`
   - **Password** : `admin123` (lub ten który utworzyłeś)

## 🔒 Bezpieczeństwo

### Zalecenia

1. **Zmienić domyślne hasło**
   ```bash
   airflow users set-password admin
   ```

2. **Używać HTTPS** (w produkcji)
   - Skonfigurować reverse proxy (nginx, Apache)
   - Używać certyfikatów SSL

3. **Ograniczać dostęp sieciowy**
   - Używać VPN
   - Ograniczać dozwolone IP w firewall

4. **Uwierzytelnianie wzmocnione**
   - Używać OAuth
   - Integrować z LDAP/Active Directory

### Konfiguracja bezpieczna

**Zmodyfikować `airflow.cfg` :**

```ini
[webserver]
# Włączyć uwierzytelnianie
auth_backend = airflow.api.auth.backend.basic_auth

# Ograniczyć dozwolone hosty (opcjonalne)
hostname_callable = airflow.utils.net.get_hostname
```

## 🐛 Rozwiązywanie problemów

### Problem : Niemożliwe połączenie z maszyny B

**Rozwiązania :**

1. **Sprawdzić firewall**
   ```bash
   # Windows
   netsh advfirewall firewall show rule name="Airflow Web Server"
   
   # Linux
   sudo ufw status
   ```

2. **Sprawdzić że Airflow nasłuchuje na 0.0.0.0**
   ```bash
   # Sprawdzić otwarte porty
   netstat -an | grep 8080
   # Powinno pokazać : 0.0.0.0:8080
   ```

3. **Sprawdzić konfigurację sieci**
   - Obie maszyny są w tej samej sieci
   - Brak VPN blokującego połączenie
   - Brak proxy interferującego

### Problem : Błąd "Connection refused"

**Rozwiązania :**

1. Sprawdzić że serwer web jest uruchomiony
2. Sprawdzić port (8080 domyślnie)
3. Sprawdzić logi Airflow :
   ```bash
   # Logi webserver
   tail -f ~/airflow/logs/webserver.log
   ```

### Problem : Błąd uwierzytelniania

**Rozwiązania :**

1. Sprawdzić credentials
2. Utworzyć ponownie użytkownika jeśli potrzeba :
   ```bash
   airflow users create \
       --username admin \
       --role Admin \
       --email admin@example.com \
       --password nowe_haslo
   ```

## 📝 Konfiguracja zaawansowana

### Używać reverse proxy (nginx)

**Instalacja nginx :**

```bash
# Linux
sudo apt install nginx

# Konfiguracja nginx
sudo nano /etc/nginx/sites-available/airflow
```

**Konfiguracja nginx :**

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

**Włączyć konfigurację :**

```bash
sudo ln -s /etc/nginx/sites-available/airflow /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### Usługa systemd (Linux)

**Utworzyć usługę dla Airflow :**

```bash
sudo nano /etc/systemd/system/airflow-webserver.service
```

**Zawartość :**

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

**Włączyć usługę :**

```bash
sudo systemctl daemon-reload
sudo systemctl enable airflow-webserver
sudo systemctl start airflow-webserver
```

## 📊 Weryfikacja

### Test połączenia

**Z maszyny B :**

```bash
# Test HTTP
curl http://192.168.1.100:8080/health

# Test z uwierzytelnianiem
curl -u admin:admin123 http://192.168.1.100:8080/api/v1/dags
```

### Sprawdzić logi

**Na maszynie A :**

```bash
# Logi webserver
tail -f ~/airflow/logs/webserver.log

# Logi scheduler
tail -f ~/airflow/logs/scheduler/*.log
```

## 🔗 Zasoby

- [Dokumentacja Airflow](https://airflow.apache.org/docs/)
- [Konfiguracja Airflow](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html)
- [Bezpieczeństwo Airflow](https://airflow.apache.org/docs/apache-airflow/stable/security/index.html)

---

**Uwaga :** Ta konfiguracja jest dla środowiska deweloperskiego. Dla produkcji, używać wzmocnionych praktyk bezpieczeństwa (HTTPS, uwierzytelnianie OAuth, itp.).

