# Airflow Installation - Remote Access

## 🎯 Objective

This guide allows you to install Apache Airflow on **Machine A** (server) and access it from **Machine B** (client) via the local network.

## 📋 Prerequisites

### Machine A (Server)
- Python 3.8+ installed
- Administrator access
- Active network connection
- Port 8080 available (or another port)

### Machine B (Client)
- Web browser
- Connection to the same local network as Machine A

## 🔧 Installation on Machine A

### Step 1: Install Python and Dependencies

**Windows:**
```powershell
# Check Python
python --version

# Install pip if needed
python -m ensurepip --upgrade
```

**Linux:**
```bash
# Install Python and pip
sudo apt update
sudo apt install python3 python3-pip python3-venv
```

### Step 2: Create Airflow Environment

```bash
# Create a directory for Airflow
mkdir airflow-install
cd airflow-install

# Create a virtual environment
python -m venv airflow-env

# Activate the environment
# Windows
airflow-env\Scripts\activate
# Linux
source airflow-env/bin/activate

# Install Airflow
pip install apache-airflow

# Install PostgreSQL provider (optional, for metadata database)
pip install apache-airflow-providers-postgres
```

### Step 3: Configure Airflow

```bash
# Initialize the database
airflow db init

# Create an administrator user
airflow users create \
    --username admin \
    --firstname Admin \
    --lastname User \
    --role Admin \
    --email admin@example.com \
    --password admin123
```

### Step 4: Configure Network Access

**Modify Airflow configuration:**

1. Find the `airflow.cfg` file:
   - Windows: `%USERPROFILE%\airflow\airflow.cfg`
   - Linux: `~/airflow/airflow.cfg`

2. Modify the following parameters:

```ini
[webserver]
# Allow access from all interfaces
web_server_host = 0.0.0.0
web_server_port = 8080

# Disable basic authentication (optional, for development)
auth_backend = airflow.api.auth.backend.basic_auth
```

**Or create a custom configuration file:**

```bash
# Create a custom airflow.cfg file
export AIRFLOW_HOME=/path/to/airflow
airflow config get-value webserver web_server_host
```

### Step 5: Configure Firewall

**Windows (Firewall):**

1. Open "Windows Defender Firewall"
2. "Advanced settings"
3. "Inbound Rules" → "New Rule"
4. Type: Port
5. Port: 8080 (TCP)
6. Action: Allow the connection
7. Name: "Airflow Web Server"

**Linux (UFW):**

```bash
# Allow port 8080
sudo ufw allow 8080/tcp
sudo ufw reload
```

**Linux (firewalld):**

```bash
# Allow port 8080
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
```

### Step 6: Start Airflow

**Terminal 1 - Web Server:**

```bash
# Activate the virtual environment
source airflow-env/bin/activate  # Linux
# or
airflow-env\Scripts\activate  # Windows

# Start the web server
airflow webserver --port 8080 --host 0.0.0.0
```

**Terminal 2 - Scheduler:**

```bash
# Activate the virtual environment
source airflow-env/bin/activate  # Linux
# or
airflow-env\Scripts\activate  # Windows

# Start the scheduler
airflow scheduler
```

### Step 7: Get Machine A IP Address

**Windows:**
```powershell
ipconfig
# Look for "IPv4 Address" (e.g., 192.168.1.100)
```

**Linux:**
```bash
ip addr show
# or
hostname -I
# Look for the IP address (e.g., 192.168.1.100)
```

## 🌐 Access from Machine B

### Step 1: Verify Connectivity

**From Machine B:**

```bash
# Test connection
ping 192.168.1.100  # Replace with Machine A IP

# Test port
telnet 192.168.1.100 8080
# or
curl http://192.168.1.100:8080
```

### Step 2: Access Web Interface

1. Open a browser on Machine B
2. Go to: `http://192.168.1.100:8080`
   - Replace `192.168.1.100` with Machine A IP
3. Log in with:
   - **Username**: `admin`
   - **Password**: `admin123` (or the one you created)

## 🔒 Security

### Recommendations

1. **Change default password**
   ```bash
   airflow users set-password admin
   ```

2. **Use HTTPS** (in production)
   - Configure a reverse proxy (nginx, Apache)
   - Use SSL certificates

3. **Limit network access**
   - Use a VPN
   - Restrict allowed IPs in firewall

4. **Enhanced authentication**
   - Use OAuth
   - Integrate with LDAP/Active Directory

### Secure Configuration

**Modify `airflow.cfg`:**

```ini
[webserver]
# Enable authentication
auth_backend = airflow.api.auth.backend.basic_auth

# Limit allowed hosts (optional)
hostname_callable = airflow.utils.net.get_hostname
```

## 🐛 Troubleshooting

### Issue: Cannot connect from Machine B

**Solutions:**

1. **Check firewall**
   ```bash
   # Windows
   netsh advfirewall firewall show rule name="Airflow Web Server"
   
   # Linux
   sudo ufw status
   ```

2. **Verify Airflow listens on 0.0.0.0**
   ```bash
   # Check open ports
   netstat -an | grep 8080
   # Should show: 0.0.0.0:8080
   ```

3. **Check network configuration**
   - Both machines are on the same network
   - No VPN blocking connection
   - No proxy interfering

### Issue: "Connection refused" error

**Solutions:**

1. Verify web server is started
2. Check the port (8080 by default)
3. Check Airflow logs:
   ```bash
   # Webserver logs
   tail -f ~/airflow/logs/webserver.log
   ```

### Issue: Authentication error

**Solutions:**

1. Verify credentials
2. Recreate user if needed:
   ```bash
   airflow users create \
       --username admin \
       --role Admin \
       --email admin@example.com \
       --password new_password
   ```

## 📝 Advanced Configuration

### Use a reverse proxy (nginx)

**Install nginx:**

```bash
# Linux
sudo apt install nginx

# Configure nginx
sudo nano /etc/nginx/sites-available/airflow
```

**Nginx configuration:**

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

**Enable configuration:**

```bash
sudo ln -s /etc/nginx/sites-available/airflow /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### Systemd service (Linux)

**Create a service for Airflow:**

```bash
sudo nano /etc/systemd/system/airflow-webserver.service
```

**Content:**

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

**Enable service:**

```bash
sudo systemctl daemon-reload
sudo systemctl enable airflow-webserver
sudo systemctl start airflow-webserver
```

## 📊 Verification

### Connection Test

**From Machine B:**

```bash
# HTTP test
curl http://192.168.1.100:8080/health

# Test with authentication
curl -u admin:admin123 http://192.168.1.100:8080/api/v1/dags
```

### Check Logs

**On Machine A:**

```bash
# Webserver logs
tail -f ~/airflow/logs/webserver.log

# Scheduler logs
tail -f ~/airflow/logs/scheduler/*.log
```

## 🔗 Resources

- [Airflow Documentation](https://airflow.apache.org/docs/)
- [Airflow Configuration](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html)
- [Airflow Security](https://airflow.apache.org/docs/apache-airflow/stable/security/index.html)

---

**Note:** This configuration is for a development environment. For production, use enhanced security practices (HTTPS, OAuth authentication, etc.).

