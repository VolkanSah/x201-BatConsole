# x201 – Web Server & Database Setup

## Table of Contents

1. [LAMP + Python Stack](#1-lamp--python-stack)
2. [Secure & Harden Databases](#2-secure--harden-databases)
3. [Python AI Environment](#3-python-ai-environment)
4. [Web Management Tools](#4-web-management-tools)
5. [Performance Optimizations](#5-performance-optimizations)
6. [Security Configuration & Tor](#6-security-configuration--tor)
7. [Finalization](#7-finalization)
8. [Useful Links](#8-useful-links)

---

## 1. LAMP + Python Stack

### Apache + PHP

```bash
sudo apt install apache2 php8.3 php8.3-fpm libapache2-mod-php8.3
````

### PHP Extensions

```bash
sudo apt install php8.3-mysql php8.3-pgsql php8.3-gd php8.3-curl \
php8.3-zip php8.3-xml php8.3-mbstring php8.3-intl php8.3-bcmath \
php8.3-imagick php8.3-opcache php8.3-apcu php8.3-sqlite3 \
php8.3-ldap php8.3-imap php8.3-soap php8.3-xmlrpc php8.3-xsl php8.3-bz2
```

### Databases

```bash
sudo apt install mariadb-server postgresql postgresql-contrib
```

### Python (for AI & Backend)

```bash
sudo apt install python3.12 python3.12-venv python3-pip
```

---

## 2. Secure & Harden Databases

### Secure MariaDB

```bash
sudo mysql_secure_installation
```

### Set PostgreSQL Password

```bash
sudo -u postgres psql
ALTER USER postgres PASSWORD 'your_strong_password';
\q
```

### Change Auth Method (example for version 16)

```bash
sudo nano /etc/postgresql/16/main/pg_hba.conf
```

Change:

```conf
local   all   postgres   peer
```

to:

```conf
local   all   postgres   md5
```

### Harden PostgreSQL Configuration

```bash
sudo nano /etc/postgresql/16/main/postgresql.conf
```

Recommended settings:

```conf
listen_addresses = 'localhost'
ssl = on
log_connections = on
log_disconnections = on
```

### Firewall Rules for Local DB Usage

```bash
sudo ufw allow from 127.0.0.1 to any port 3306  # MariaDB
sudo ufw allow from 127.0.0.1 to any port 5432  # PostgreSQL
```

> **Ask yourself:** If Tor listens on `localhost` – do you really want to allow DB access via hidden services? Only recommended for dedicated setups with full Auth & ACL.

---

## 3. Python AI Environment

### Virtual Environment

```bash
python3 -m venv ~/ai-env
source ~/ai-env/bin/activate
```

### Install Libraries

```bash
pip install numpy pandas scikit-learn matplotlib jupyter
pip install torch torchvision transformers datasets
```

---

## 4. Web Management Tools

| Tool    | Installation                                                           | Port |
| ------- | ---------------------------------------------------------------------- | ---- |
| Adminer | `wget -O /var/www/html/adminer.php https://www.adminer.org/latest.php` | 80   |
| Jupyter | `pip install jupyter` (runs in `~/ai-env`)                             | 8888 |

---

## 5. Performance Optimizations

### Enable Apache Modules

```bash
sudo a2enmod proxy_fcgi setenvif headers expires deflate http2 rewrite ssl
```

### Enable PHP-FPM Config

```bash
sudo a2enconf php8.3-fpm
```

### Adjust Opcache

```ini
# /etc/php/8.3/fpm/php.ini
opcache.memory_consumption=128
opcache.max_accelerated_files=4000
```

---

## 6. Security Configuration & Tor

### Setup Tor Hidden Service

```bash
sudo apt install tor
sudo nano /etc/tor/torrc
```

Example configuration:

```conf
HiddenServiceDir /var/lib/tor/hidden_service/
HiddenServicePort 80 127.0.0.1:80
```

> Warning: Never link services in Hidden Service directly to PostgreSQL/MariaDB without strong authentication!

### Additional Security Packages

```bash
sudo apt install fail2ban ufw modsecurity
```

### More Tools (optional)

```bash
sudo apt install imagemagick redis-server php8.3-redis memcached php8.3-memcached \
ffmpeg ghostscript webp certbot
```

---

## 7. Finalization

```bash
sudo systemctl restart apache2 mariadb postgresql
sudo ufw enable
```
```
sudo ufw default deny incoming      # Alles incoming dichtmachen  
sudo ufw default allow outgoing     # Outgoing offen lassen (meistens sinnvoll)  
sudo ufw allow from 192.168.3.0/24 to any port 22  # SSH aus lokalem Netz erlauben  
sudo ufw allow 80                   # HTTP  
sudo ufw allow 443                  # HTTPS  

```


## 8. Useful Links

* [Apache Performance Tuning](https://httpd.apache.org/docs/2.4/misc/perf-tuning.html)
* [PostgreSQL Security Docs](https://www.postgresql.org/docs/current/security.html)
* [Tor Project Support](https://support.torproject.org/)



