**LAMP + Python Stack:**
```bash
# Apache + PHP
sudo apt install apache2 php8.3 php8.3-fpm libapache2-mod-php8.3

# PHP Extensions
sudo apt install php8.3-mysql php8.3-pgsql php8.3-gd php8.3-curl php8.3-zip php8.3-xml php8.3-mbstring php8.3-intl php8.3-bcmath php8.3-imagick php8.3-opcache

# Datenbanken
sudo apt install mariadb-server postgresql postgresql-contrib

# Python für KI/ML
sudo apt install python3.12 python3.12-venv python3-pip
```

# Datenbanken sichern: 
Maria Db
```
sudo mysql_secure_installation
```
Postgree bissel tricky

Passwort setzten:
```sudo -u postgres psql

# 1. postgres user (= root) starkes Password geben
ALTER USER postgres PASSWORD 'dein_starkes_postgres_password';
# Achtung Keine Sonderzeuchen! 

# 2. Raus aus psql
\q
```
exampl for version 16:

```
sudo nano /etc/postgresql/16/main/pg_hba.conf
```
das: 
```
local   all             postgres                                peer
```
zu das ändern:
```
local   all             postgres                                md5
```
sudo systemctl restart postgresql
# Jetzt sollte nach Password gefragt werden
sudo -u postgres psql


**Für Python KI-Libs:**
```bash
# Virtual Environment erstellen
python3 -m venv ~/ai-env
source ~/ai-env/bin/activate

# Essentials
pip install numpy pandas scikit-learn matplotlib jupyter
pip install torch torchvision # PyTorch (CPU-only für X201)
pip install transformers datasets # Hugging Face
```

**Web-Management:**
- **Adminer:** `wget https://www.adminer.org/latest.php` nach `/var/www/html/`
- **Jupyter:** läuft auf Port 8888, perfekt für AI-Experimente



**Performance-Tweaks für X201:**
```bash
# PHP-FPM aktivieren
sudo a2enmod proxy_fcgi setenvif
sudo a2enconf php8.3-fpm

# Opcache config in php.ini
opcache.memory_consumption=128
opcache.max_accelerated_files=4000
```

Ah stimmt, die wichtigen Sachen hab ich unterschlagen! Hier die komplette Liste:

**Fehlende essentials:**
```bash
# ImageMagick + PHP Extension
sudo apt install imagemagick php8.3-imagick

# Caching
sudo apt install memcached php8.3-memcached redis-server php8.3-redis

# Media Processing
sudo apt install ffmpeg ghostscript # für PDF/Video handling
sudo apt install webp # moderne Bildformate

# Security & Performance
sudo apt install fail2ban ufw
sudo apt install php8.3-apcu # User cache für PHP
```

**Für Journalisten-CMS + Tor:**
```bash
# Tor Hidden Service
sudo apt install tor
# In /etc/tor/torrc:
# HiddenServiceDir /var/lib/tor/hidden_service/
# HiddenServicePort 80 127.0.0.1:80

# SSL/TLS auch für .onion (empfohlen)
sudo apt install certbot # für Let's Encrypt

# Database Performance
sudo apt install php8.3-sqlite3 # falls SQLite als Cache/Session store
```

**Zusätzliche PHP-Extensions für CMS:**
```bash
php8.3-ldap php8.3-imap php8.3-soap php8.3-xmlrpc php8.3-xsl php8.3-bz2
```

**Apache Module für Performance:**
```bash
sudo a2enmod headers expires deflate http2 rewrite ssl
```

**Security für Journalisten-CMS:**
- ModSecurity WAF
- Rate limiting
- Log anonymization
- Database encryption at rest

