# x201 – Webserver & Datenbanken Setup

## Inhaltsverzeichnis

1. [LAMP + Python Stack](#1-lamp--python-stack)
2. [Datenbanken sichern & härten](#2-datenbanken-sichern--härten)
3. [Python KI-Umgebung](#3-python-ki-umgebung)
4. [Web-Management-Tools](#4-web-management-tools)
5. [Performance-Optimierungen](#5-performance-optimierungen)
6. [Sicherheitskonfiguration & Tor](#6-sicherheitskonfiguration--tor)
7. [Fertigstellung](#7-fertigstellung)
8. [Nützliche Links](#8-nützliche-links)

---

## 1. LAMP + Python Stack

### Apache + PHP

```bash
sudo apt install apache2 php8.3 php8.3-fpm libapache2-mod-php8.3
```

### PHP-Erweiterungen

```bash
sudo apt install php8.3-mysql php8.3-pgsql php8.3-gd php8.3-curl \
php8.3-zip php8.3-xml php8.3-mbstring php8.3-intl php8.3-bcmath \
php8.3-imagick php8.3-opcache php8.3-apcu php8.3-sqlite3 \
php8.3-ldap php8.3-imap php8.3-soap php8.3-xmlrpc php8.3-xsl php8.3-bz2
```

### Datenbanken

```bash
sudo apt install mariadb-server postgresql postgresql-contrib
```

### Python (für AI & Backend)

```bash
sudo apt install python3.12 python3.12-venv python3-pip
```

---

## 2. Datenbanken sichern & härten

### MariaDB absichern

```bash
sudo mysql_secure_installation
```

### PostgreSQL Passwort setzen

```bash
sudo -u postgres psql
ALTER USER postgres PASSWORD 'dein_starkes_passwort';
\q
```

### Auth-Methode ändern (Beispiel für Version 16)

```bash
sudo nano /etc/postgresql/16/main/pg_hba.conf
```

Ändern:

```conf
local   all   postgres   peer
```

zu:

```conf
local   all   postgres   md5
```

### PostgreSQL-Konfiguration härten

```bash
sudo nano /etc/postgresql/16/main/postgresql.conf
```

Empfohlene Einstellungen:

```conf
listen_addresses = 'localhost'
ssl = on
log_connections = on
log_disconnections = on
```

### Firewall-Regeln für lokale DB-Nutzung

```bash
sudo ufw allow from 127.0.0.1 to any port 3306  # MariaDB
sudo ufw allow from 127.0.0.1 to any port 5432  # PostgreSQL
```

> **Frage an dich selbst:** Wenn Tor auf `localhost` lauscht – willst du Datenbankzugriffe wirklich über Hidden Services erlauben? Nur für dedizierte Setups mit voller Auth & ACL zu empfehlen.

---

## 3. Python KI-Umgebung

### Virtuelle Umgebung

```bash
python3 -m venv ~/ai-env
source ~/ai-env/bin/activate
```

### Bibliotheken installieren

```bash
pip install numpy pandas scikit-learn matplotlib jupyter
pip install torch torchvision transformers datasets
```

---

## 4. Web-Management-Tools

| Tool    | Installation                                                           | Port |
| ------- | ---------------------------------------------------------------------- | ---- |
| Adminer | `wget -O /var/www/html/adminer.php https://www.adminer.org/latest.php` | 80   |
| Jupyter | `pip install jupyter` (läuft in `~/ai-env`)                            | 8888 |

---

## 5. Performance-Optimierungen

### Apache Module aktivieren

```bash
sudo a2enmod proxy_fcgi setenvif headers expires deflate http2 rewrite ssl
```

### PHP-FPM Konfiguration aktivieren

```bash
sudo a2enconf php8.3-fpm
```

### Opcache anpassen

```ini
# /etc/php/8.3/fpm/php.ini
opcache.memory_consumption=128
opcache.max_accelerated_files=4000
```

---

## 6. Sicherheitskonfiguration & Tor

### Tor Hidden Service einrichten

```bash
sudo apt install tor
sudo nano /etc/tor/torrc
```

Beispiel-Konfiguration:

```conf
HiddenServiceDir /var/lib/tor/hidden_service/
HiddenServicePort 80 127.0.0.1:80
```

> Achtung: Dienste im Hidden Service nie direkt auf PostgreSQL/MariaDB verlinken ohne starke Authentifizierung!

### Zusätzliche Sicherheitspakete

```bash
sudo apt install fail2ban ufw modsecurity
```

### Weitere Tools (optional)

```bash
sudo apt install imagemagick redis-server php8.3-redis memcached php8.3-memcached \
ffmpeg ghostscript webp certbot
```

---

## 7. Fertigstellung

```bash
sudo systemctl restart apache2 mariadb postgresql
sudo ufw enable
```

---

## 8. Nützliche Links

* [Apache Performance Tuning](https://httpd.apache.org/docs/2.4/misc/perf-tuning.html)
* [PostgreSQL Security Docs](https://www.postgresql.org/docs/current/security.html)
* [Tor Project Support](https://support.torproject.org/)

