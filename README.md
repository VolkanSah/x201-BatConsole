# 🖥️ x201 – BatConsole Setup & Tuning Guide

Was tun mit einem alten Lenovo X201?  
→ Einen flüsterleisen, stabilen und WLAN-fähigen Entwickler-/Heimserver bauen! 🦇

Wieso Ubuntu und nicht Debian? [Hier](why-ubuntu.md)

---

## 📋 Inhaltsübersicht
1. [Systemoptimierung & Kühlung](#-1-systemoptimierung--kühlung)
2. [BIOS-Tuning](#-2-bios-tuning)
3. [Lüfter & Last-Test](#-3-lüfter--last-test-optional)
4. [WLAN-Setup](#-4-wlan-setup-für-serverbetrieb)
5. [Sicherheit](#-5-grundschutz--sicherheit)

---

## ⚙️ 1. Systemoptimierung & Kühlung

### 🔋 TLP für Akku & CPU-Optimierung
```bash
sudo apt install tlp tlp-rdw
sudo systemctl enable tlp --now
```

### 🧠 CPU-Governor setzen
```bash
sudo apt install cpufrequtils
echo 'GOVERNOR="powersave"' | sudo tee /etc/default/cpufrequtils
sudo systemctl restart cpufrequtils
```

#### 🔄 Dauerhafte Einstellung (empfohlen)
```bash
echo 'GOVERNOR="conservative"' | sudo tee /etc/default/cpufrequtils
sudo systemctl restart cpufrequtils
```

### 🌡️ Temperaturüberwachung
```bash
sudo apt install lm-sensors
sudo sensors-detect
watch -n 1 "sensors && cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor"
```

#### 📊 CPU-Frequenz anzeigen
```bash
watch -n 1 "grep 'MHz' /proc/cpuinfo"
```

---

## 🧬 2. BIOS-Tuning
- ✅ Hyper-Threading **aktivieren**
- ❌ Intel Turbo Boost **deaktivieren** (bei Überhitzung)
- 🔧 Lüftersteuerung auf "Performance" (falls verfügbar)

---

## 🌀 3. Lüfter & Last-Test (optional)
```bash
sudo apt install fancontrol pwmconfig stress s-tui
sudo pwmconfig        # Achtung bei Laptops!
stress --cpu 4        # Anzahl der Kerne anpassen
s-tui                 # Echtzeit-Monitoring
```

---

## 📶 4. WLAN-Setup für Serverbetrieb

### 📂 WPA-Konfiguration (`/etc/wpa_supplicant/wpa_supplicant.conf`)
```conf
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=DE

network={
    ssid="DeinWLANName"
    psk="DeinPasswort"
    key_mgmt=WPA-PSK
}
```

### 🌐 Netplan Konfiguration (`/etc/netplan/01-netcfg.yaml`)
```yaml
network:
  version: 2
  renderer: networkd
  wifis:
    wlp2s0:
      dhcp4: true
      access-points:
        "DeinWLANName":
          password: "DeinPasswort"
```

#### 🔒 Berechtigungen setzen
```bash
sudo chmod 600 /etc/netplan/01-netcfg.yaml
sudo netplan apply
```

### 🧪 WLAN-Verbindung testen
```bash
iw dev wlp2s0 link                # Verbindungsstatus
ip route | grep default           # Aktive Schnittstelle
ping -I wlp2s0 8.8.8.8            # WLAN-Pingtest
```

---

## 🔒 5. Grundschutz & Sicherheit

### 🛠️ System-Härtung
```bash
# Dash als /bin/sh deaktivieren (für Kompatibilität)
sudo dpkg-reconfigure dash
# Wähle "Nein" bei der Frage nach Standard-System-Shell

# AppArmor deaktivieren (falls nicht benötigt)
sudo systemctl stop apparmor
sudo systemctl disable apparmor
sudo apt purge apparmor apparmor-utils -y
```

### 🦠 ClamAV (Virenscanner)
```bash
sudo apt install clamav clamav-daemon -y
sudo systemctl enable clamav-freshclam --now
sudo freshclam
```

### 🕵️ chkrootkit (Rootkit-Erkennung)
```bash
sudo apt install chkrootkit -y
sudo chkrootkit
```

### 🔍 rkhunter (Erweiterte Rootkit-Erkennung)
```bash
sudo apt install rkhunter -y
sudo rkhunter --update
sudo rkhunter --propupd
sudo rkhunter --check
```

### 🛡️ fail2ban (Brute-Force-Schutz)
```bash
sudo apt install fail2ban -y
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

#### Beispielkonfiguration für SSH-Härtung (`/etc/fail2ban/jail.local`):
```ini
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
findtime = 600
```

#### Konfiguration anwenden:
```bash
sudo systemctl restart fail2ban
```

### 📊 Statuschecks
```bash
# Alle Sicherheitsdienste überprüfen
sudo systemctl status clamav-daemon
sudo fail2ban-client status
sudo rkhunter --check --sk
```

---

## 🎉 Fertigstellung
Dein X201 ist nun optimiert und abgesichert. Starte das System neu, um alle Änderungen zu aktivieren:

```bash
sudo reboot
```

> **Tipp:** Nach dem Neustart die Temperatur mit `sensors` überprüfen und die Sicherheitsdienste testen.

---

## 🔗 Nützliche Links
- [Ubuntu Server Guide](https://ubuntu.com/server/docs)
- [ThinkPad Hardware-Support](https://www.thinkwiki.org)
- [Linux Security Hardening](https://linuxsecurity.com/features)
