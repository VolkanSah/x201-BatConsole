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
or
```
sudo systemctl restart ssh.service
```

**Validation**
```bash
sudo sshd -t
grep -E "^(PermitRootLogin|PasswordAuthentication|Port)" /etc/ssh/sshd_config
```

## Kernel Security (sysctl)

Harden your Linux kernel security by configuring sysctl parameters. You can either add rules directly to /etc/sysctl.conf or, preferably, create /etc/sysctl.d/99-security.conf and input [this rules](etc/sysctl.conf) there.

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
# First check what accounts exist and their purpose
cat /etc/passwd | grep -E "nobody|daemon|bin|sys|sync|games|man|lp|mail|news|uucp|proxy|www-data|backup|list|irc"

# Only disable accounts you actually don't need
# Common candidates for most setups:
sudo usermod -L -s /bin/false games
sudo usermod -L -s /bin/false news
sudo usermod -L -s /bin/false uucp
sudo usermod -L -s /bin/false irc

# CAREFUL - these might be needed depending on your services:
# www-data (web server - DO NOT disable if running Apache/Nginx)
# mail (mail services)
# proxy (proxy services) 
# backup (backup scripts)
# nobody (some services use this)
# daemon (system services)
```

**Validation**
```bash
cat /etc/passwd | grep -E "nobody|daemon|bin|sys|sync|games|man|lp|mail|news|uucp|proxy|www-data|backup|list|irc"
sudo -l
```


## Firewall Configuration

If you've already configured your firewall rules according to this guide, you do not need to rewrite them. Refer to the [Basic Firewall Rules (UFW) for Local & Secure Server Setups](https://github.com/VolkanSah/x201-BatConsole/blob/VolkanSah-patch-1/cap-2.md#basic-firewall-rules-ufw-for-local--secure-server-setups) for the recommended setup.

**UFW Basic Setup for Public Servers**

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

# Basic Security headers

Header always set X-Content-Type-Options nosniff
Header always set X-Frame-Options DENY
Header always set X-XSS-Protection "1; mode=block"
Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"


# Disable server status

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
curl -I http://127.0.0.1 | grep -E "Server:|X-"
apache2ctl -t
```
##### Advanced security tipps (be carful!)
- [ModSecurity Webserver Protection Guide](https://github.com/VolkanSah/ModSecurity-Webserver-Protection-Guide)
- [Security Headers Explained](https://github.com/VolkanSah/Security-Headers)

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

--- 
## 🏁 You made it!

Thanks for surviving this beautifully chaotic tutorial.  
If you learned something, laughed once, or just feel 2% smarter — drop a ⭐.

If you’re planning to fork it, test it, break it — even better.

Remember:  
**You are not secure. You are just less vulnerable than yesterday.**



## 🤖 Special thanks

Big shoutout to all the AIs who helped debug, sort data, and keep me (mostly) sane during this ride —  
even when they occasionally dumped more output than the poor server could handle.  
Still, without you: this repo would be half the madness it is now. Respect.



## ☕ Wanna say thanks?

If you ever get rich:  
Three coffees would be cool.  
If not — respect. We're in the same Batboat.



### 📚 Chapters
- [Cap-1: HomeBase](README.md.md)  
- [Cap-2: x201 – Web Server & Database Setup](cap-2.md)  
- [Cap-3: Performance & Resilience Test (Tor Edition)](cap-3.md)  
- [Cap-4: Security Audit Suite for Tor Edition Systems](cap-4.md)  
- [Cap-5: Server Hardening](cap-5.5md)  
- [Why Ubuntu and not Debian?](why-ubuntu.md)  
- [Why Apache and not NGINX?](why-apache.md)

