# 🖥️ x201 – Webserver & Datenbanken Setup

## 📋 Inhaltsverzeichnis
1. [LAMP + Python Stack](#1-lamp--python-stack)
2. [Datenbanken sichern & härten](#2-datenbanken-sichern--härten)
3. [Python KI-Umgebung](#3-python-ki-umgebung)
4. [Web Management-Tools](#4-web-management-tools)
5. [Performance-Optimierungen](#5-performance-optimierungen)
6. [Sicherheitskonfiguration](#6-sicherheitskonfiguration)

---

## 1. LAMP + Python Stack

### Apache & PHP Installation
```bash
sudo apt install apache2 php8.3 php8.3-fpm libapache2-mod-php8.3
```

### PHP Erweiterungen
```bash
sudo apt install php8.3-mysql php8.3-pgsql php8.3-gd php8.3-curl \
php8.3-zip php8.3-xml php8.3-mbstring php8.3-intl php8.3-bcmath \
php8.3-imagick php8.3-opcache
```

### Datenbanken
```bash
sudo apt install mariadb-server postgresql postgresql-contrib
```

### Python Umgebung
```bash
sudo apt install python3.12 python3.12-venv python3-pip
```

---

## 2. Datenbanken sichern & härten

### MariaDB Sicherheit
```bash
sudo mysql_secure_installation
```

### PostgreSQL Härtung
```bash
sudo -u postgres psql -c "ALTER USER postgres PASSWORD 'starkes_passwort';"
sudo nano /etc/postgresql/16/main/pg_hba.conf
```

**Empfohlene Einstellungen:**
```conf
# postgresql.conf
listen_addresses = 'localhost'
ssl = on

# pg_hba.conf
local   all   all   md5
host    all   all   127.0.0.1/32   md5
```

### Firewall-Regeln
```bash
sudo ufw allow from 127.0.0.1 to any port 3306  # MariaDB
sudo ufw allow from 127.0.0.1 to any port 5432  # PostgreSQL
```

---

## 3. Python KI-Umgebung

### Virtuelle Umgebung erstellen
```bash
python3 -m venv ~/ai-env
source ~/ai-env/bin/activate
```

### KI-Bibliotheken installieren
```bash
pip install numpy pandas scikit-learn matplotlib jupyter
pip install torch torchvision transformers datasets
```

---

## 4. Web Management-Tools

| Tool | Installationsbefehl | Port |
|------|---------------------|------|
| **Adminer** | `wget -O /var/www/html/adminer.php https://www.adminer.org/latest.php` | 80 |
| **Jupyter** | `pip install jupyter` | 8888 |

---

## 5. Performance-Optimierungen

### Apache Tweaks
```bash
sudo a2enmod proxy_fcgi setenvif headers expires deflate http2 rewrite ssl
sudo a2enconf php8.3-fpm
```

### PHP OPcache
```ini
; /etc/php/8.3/fpm/php.ini
opcache.memory_consumption=128
opcache.max_accelerated_files=4000
```

---

## 6. Sicherheitskonfiguration

### Tor Hidden Service
```bash
sudo apt install tor
echo "HiddenServiceDir /var/lib/tor/hidden_service/
HiddenServicePort 80 127.0.0.1:80" | sudo tee -a /etc/tor/torrc
```

### Wichtige Sicherheitsregeln:
1. **Niemals** Datenbanken direkt über Tor freigeben
2. Immer Firewall für `127.0.0.1` beschränken
3. Regelmäßige Backups durchführen

### Zusätzliche Sicherheitspakete
```bash
sudo apt install fail2ban ufw modsecurity-crs
```

---

##  Fertigstellung
```bash
sudo systemctl restart apache2 mariadb postgresql
sudo ufw enable
```

> **Hinweis:** Nach der Konfiguration alle Dienste neu starten und die Firewall aktivieren.

---

## 🔗 Nützliche Links
- [Apache Performance Tuning](https://httpd.apache.org/docs/2.4/misc/perf-tuning.html)
- [PostgreSQL Security](https://www.postgresql.org/docs/current/security.html)
- [Tor Project Documentation](https://support.torproject.org/)
