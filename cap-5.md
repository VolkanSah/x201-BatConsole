# Chapter 5: Server Hardening

## Warning

This is not a security guarantee. This is damage reduction. You are responsible for your own infrastructure. These configurations will break things. Test before production. No support provided.

## SSH Hardening

**Disable root login and password authentication**
```bash
# Edit /etc/ssh/sshd_config
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
Protocol 2
Port 2222
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 2
AllowUsers yourusername
```

**Restart SSH service**
```bash
sudo systemctl restart sshd
```

**Validation**
```bash
sudo sshd -t
grep -E "^(PermitRootLogin|PasswordAuthentication|Port)" /etc/ssh/sshd_config
```

## Kernel Security (sysctl)

**Create /etc/sysctl.d/99-security.conf**
```bash
# IP Spoofing protection
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.all.rp_filter = 1

# Ignore ICMP redirects
net.ipv4.conf.all.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0

# Ignore send redirects
net.ipv4.conf.all.send_redirects = 0

# Disable source packet routing
net.ipv4.conf.all.accept_source_route = 0
net.ipv6.conf.all.accept_source_route = 0

# Log Martians
net.ipv4.conf.all.log_martians = 1

# Ignore ping requests
net.ipv4.icmp_echo_ignore_all = 1

# Ignore broadcast requests
net.ipv4.icmp_echo_ignore_broadcasts = 1

# SYN flood protection
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 2048
net.ipv4.tcp_synack_retries = 2
net.ipv4.tcp_syn_retries = 5
```

**Apply settings**
```bash
sudo sysctl -p /etc/sysctl.d/99-security.conf
```

**Validation**
```bash
sudo sysctl -a | grep -E "rp_filter|accept_redirects|send_redirects|accept_source_route|log_martians|icmp_echo_ignore|tcp_syncookies"
```

## User Privileges and sudo

**Lock down sudo access**
```bash
# Edit /etc/sudoers with visudo
sudo visudo

# Add specific commands only
username ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx, /usr/bin/systemctl status nginx
```

**Disable unused accounts**
```bash
sudo usermod -L -s /bin/false nobody
sudo usermod -L -s /bin/false daemon
sudo usermod -L -s /bin/false bin
sudo usermod -L -s /bin/false sys
sudo usermod -L -s /bin/false sync
sudo usermod -L -s /bin/false games
sudo usermod -L -s /bin/false man
sudo usermod -L -s /bin/false lp
sudo usermod -L -s /bin/false mail
sudo usermod -L -s /bin/false news
sudo usermod -L -s /bin/false uucp
sudo usermod -L -s /bin/false proxy
sudo usermod -L -s /bin/false www-data
sudo usermod -L -s /bin/false backup
sudo usermod -L -s /bin/false list
sudo usermod -L -s /bin/false irc
```

**Validation**
```bash
cat /etc/passwd | grep -E "nobody|daemon|bin|sys|sync|games|man|lp|mail|news|uucp|proxy|www-data|backup|list|irc"
sudo -l
```

## Firewall Configuration

**UFW basic setup**
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2222/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

**Advanced iptables rules**
```bash
# Drop invalid packets
sudo iptables -A INPUT -m state --state INVALID -j DROP

# Rate limit SSH connections
sudo iptables -A INPUT -p tcp --dport 2222 -m state --state NEW -m recent --set
sudo iptables -A INPUT -p tcp --dport 2222 -m state --state NEW -m recent --update --seconds 60 --hitcount 4 -j DROP

# Log dropped packets
sudo iptables -A INPUT -j LOG --log-prefix "DROPPED: "
```

**Validation**
```bash
sudo ufw status verbose
sudo iptables -L -n -v
```

## Apache Security

**Edit /etc/apache2/conf-available/security.conf**
```apache
# Hide version information
ServerTokens Prod
ServerSignature Off

# Security headers
Header always set X-Content-Type-Options nosniff
Header always set X-Frame-Options DENY
Header always set X-XSS-Protection "1; mode=block"
Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
Header always set Content-Security-Policy "default-src 'self'"

# Disable server status
<Location "/server-status">
    Require all denied
</Location>

<Location "/server-info">
    Require all denied
</Location>
```

**Enable security configuration**
```bash
sudo a2enconf security
sudo systemctl reload apache2
```

**Hide sensitive files**
```apache
# Add to .htaccess or apache config
<Files ~ "^\.">
    Require all denied
</Files>

<Files ~ "(\.bak|\.config|\.sql|\.log)$">
    Require all denied
</Files>
```

**Validation**
```bash
curl -I http://localhost | grep -E "Server:|X-"
apache2ctl -t
```

## Logging and Monitoring

**Configure rsyslog for centralized logging**
```bash
# Edit /etc/rsyslog.conf
*.info;mail.none;authpriv.none;cron.none    /var/log/messages
authpriv.*                                  /var/log/secure
mail.*                                      /var/log/maillog
cron.*                                      /var/log/cron
*.emerg                                     *
```

**Set up logrotate**
```bash
# Create /etc/logrotate.d/security
/var/log/secure {
    weekly
    rotate 4
    compress
    delaycompress
    missingok
    notifempty
    create 0600 root root
}
```

**Basic intrusion detection**
```bash
# Install fail2ban
sudo apt install fail2ban

# Create /etc/fail2ban/jail.local
[sshd]
enabled = true
port = 2222
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600

[apache-auth]
enabled = true
filter = apache-auth
logpath = /var/log/apache2/error.log
maxretry = 3
bantime = 3600
```

**Validation**
```bash
sudo systemctl status rsyslog
sudo systemctl status fail2ban
sudo fail2ban-client status
tail -f /var/log/auth.log
```

## File System Security

**Set proper permissions**
```bash
# Secure /etc/passwd and /etc/shadow
sudo chmod 644 /etc/passwd
sudo chmod 600 /etc/shadow

# Secure SSH keys
sudo chmod 700 ~/.ssh
sudo chmod 600 ~/.ssh/authorized_keys

# Secure web directories
sudo chmod 755 /var/www
sudo chmod 644 /var/www/html/*
sudo chown -R www-data:www-data /var/www/html
```

**Remove unnecessary packages**
```bash
sudo apt autoremove
sudo apt purge telnet ftp rsh-client rsh-redone-client
```

**Validation**
```bash
ls -la /etc/passwd /etc/shadow
ls -la ~/.ssh/
find /var/www -type f -exec ls -la {} \;
```

## Network Security

**Disable unused services**
```bash
sudo systemctl disable cups
sudo systemctl disable avahi-daemon
sudo systemctl disable bluetooth
sudo systemctl stop cups
sudo systemctl stop avahi-daemon
sudo systemctl stop bluetooth
```

**Check listening ports**
```bash
sudo netstat -tulpn | grep LISTEN
sudo ss -tulpn | grep LISTEN
```

**Validation**
```bash
sudo systemctl list-units --type=service --state=running
nmap -sS -O localhost
```

## Manual Verification Points

Open your SSH config. If you find `PermitRootLogin yes`, you failed.

Check your firewall. If you see `Status: inactive`, you failed.

Review your Apache headers. If you see full version information, you failed.

Examine your user accounts. If daemon users have login shells, you failed.

Check your kernel parameters. If IP forwarding is enabled without purpose, you failed.

Review your log files. If they are not being rotated, you failed.

**Final validation command**
```bash
# Run this comprehensive check
echo "=== SSH Config ==="
grep -E "^(PermitRootLogin|PasswordAuthentication|Port)" /etc/ssh/sshd_config

echo "=== Firewall Status ==="
sudo ufw status

echo "=== Listening Services ==="
sudo ss -tulpn | grep LISTEN

echo "=== Failed Login Attempts ==="
sudo grep "Failed password" /var/log/auth.log | tail -5

echo "=== System Updates ==="
apt list --upgradable
```

You are not secure. You are just less vulnerable than yesterday.
