# 2. Webserver & Datenbanken

## Inhaltsverzeichnis

1. LAMP + Python Stack
2. Datenbanken sichern & härten
3. Python KI-Umgebung
4. Web Management-Tools
5. Performance-Tweaks für X201
6. Fehlende Essentials
7. Journalisten-CMS + Tor
8. Apache Performance-Module
9. CMS-Security
10. Hinweis zu Tor & Datenbanksicherheit

---

## 1. LAMP + Python Stack

```bash
# Apache + PHP
sudo apt install apache2 php8.3 php8.3-fpm libapache2-mod-php8.3

# PHP Extensions
sudo apt install php8.3-mysql php8.3-pgsql php8.3-gd php8.3-curl \
php8.3-zip php8.3-xml php8.3-mbstring php8.3-intl php8.3-bcmath \
php8.3-imagick php8.3-opcache

# Datenbanken
sudo apt install mariadb-server postgresql postgresql-contrib

# Python für KI/ML
sudo apt install python3.12 python3.12-venv python3-pip
```

---

## 2. Datenbanken sichern & härten

### MariaDB

```bash
sudo mysql_secure_installation
```

### PostgreSQL

```bash
# Passwort setzen:
sudo -u postgres psql
ALTER USER postgres PASSWORD 'dein_starkes_postgres_password';
\q

# Auth-Methode anpassen (z. B. Version 16):
sudo nano /etc/postgresql/16/main/pg_hba.conf
# Ändere:
local   all   postgres   peer
# zu:
local   all   postgres   md5

sudo systemctl restart postgresql
```

### PostgreSQL-Konfiguration härten

```conf
# /etc/postgresql/16/main/postgresql.conf
listen_addresses = 'localhost'
ssl = on
log_connections = on
log_disconnections = on

# /etc/postgresql/16/main/pg_hba.conf
local   all   all   md5
host    all   all   127.0.0.1/32   md5
```

### Firewall für lokale DB-Verbindungen

```bash
sudo ufw allow from 127.0.0.1 to any port 3306  # MariaDB
sudo ufw allow from 127.0.0.1 to any port 5432  # PostgreSQL
sudo systemctl restart mariadb postgresql
```

---

## 3. Python KI-Umgebung

```bash
python3 -m venv ~/ai-env
source ~/ai-env/bin/activate

pip install numpy pandas scikit-learn matplotlib jupyter
pip install torch torchvision
pip install transformers datasets
```

---

## 4. Web Management-Tools

* **Adminer:** `wget https://www.adminer.org/latest.php` nach `/var/www/html/`
* **Jupyter:** läuft auf Port 8888

---

## 5. Performance-Tweaks für X201

```bash
sudo a2enmod proxy_fcgi setenvif
sudo a2enconf php8.3-fpm

# /etc/php/8.3/fpm/php.ini:
opcache.memory_consumption=128
opcache.max_accelerated_files=4000
```

---

## 6. Fehlende Essentials

```bash
sudo apt install imagemagick php8.3-imagick
sudo apt install memcached php8.3-memcached redis-server php8.3-redis
sudo apt install ffmpeg ghostscript webp
sudo apt install fail2ban ufw php8.3-apcu
```

---

## 7. Journalisten-CMS + Tor

```bash
sudo apt install tor

# /etc/tor/torrc:
HiddenServiceDir /var/lib/tor/hidden_service/
HiddenServicePort 80 127.0.0.1:80

sudo apt install certbot  # TLS auch für .onion
sudo apt install php8.3-sqlite3
```

---

## 8. Apache Performance-Module

```bash
sudo a2enmod headers expires deflate http2 rewrite ssl
```

---

## 9. CMS Security Best Practices

* ModSecurity WAF
* Rate Limiting
* Log-Anonymisierung
* Datenbankverschlüsselung (at rest)

---

## 10. Hinweis: Datenbanken & Tor Hidden Services

**Tor Hidden Services sind sicher**, aber **nie direkt Datenbanken über .onion freigeben!**

* Nur Webserver via `.onion` freigeben
* Datenbank über `localhost` verwenden
* Niemals PostgreSQL oder MariaDB als Hidden Service konfigurieren
* Firewall nur für `127.0.0.1` freigeben

> Mehr dazu siehe Abschnitt "Tor & Datenbanksicherheit"
