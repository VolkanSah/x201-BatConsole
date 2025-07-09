# 🖥️ x201 – BatConsole Setup & Tuning Guide

Was tun mit einem alten Lenovo X201?  
→ Einen flüsterleisen, stabilen und sogar WLAN-fähigen Entwickler- oder Heimserver draus bauen! 🦇

---

## ⚙️ 1. Systemoptimierung & Kühlung

### 🔋 TLP für Akku & CPU-Optimierung

```bash
sudo apt install tlp tlp-rdw
sudo systemctl enable tlp --now
````

---

### 🧠 CPU-Governor setzen

```bash
sudo apt install cpufrequtils
echo 'GOVERNOR="powersave"' | sudo tee /etc/default/cpufrequtils
sudo systemctl restart cpufrequtils
```

**Empfehlung für Dauerbetrieb:**

```bash
sudo cpufreq-set -g conservative
```

Oder dauerhaft:

```bash
echo 'GOVERNOR="conservative"' | sudo tee /etc/default/cpufrequtils
sudo systemctl restart cpufrequtils
```

---

### 🌡️ Temperatur & Throttling live checken

```bash
sudo apt install lm-sensors
sudo sensors-detect
watch -n 1 "sensors && cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor"
```

**CPU MHz-Anzeige:**

```bash
watch -n 1 "grep 'MHz' /proc/cpuinfo"
```

---

## 🧬 2. BIOS-Tuning

* ✅ Hyper-Threading **an**
* ❌ Intel Turbo Boost **aus**, wenn zu heiß
* 🔧 Lüftersteuerung auf „Performance“ (falls möglich)

---

## 🌀 3. Lüfter & Last-Test (optional)

```bash
sudo apt install fancontrol pwmconfig stress s-tui
sudo pwmconfig        # Vorsicht bei Laptops
stress --cpu 4        # Je nach Kernanzahl
s-tui                 # CPU Load & Temp in Echtzeit
```

---

## 📶 4. WLAN-Setup für Serverbetrieb

### A) WPA-Konfiguration (`/etc/wpa_supplicant/wpa_supplicant.conf`)

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

---

### B) Netplan WLAN aktivieren (`/etc/netplan/01-netcfg.yaml`)

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

**Wichtig:**
Fixe die Berechtigungen:

```bash
sudo chmod 600 /etc/netplan/01-netcfg.yaml
sudo netplan apply
```

---

## 🧪 Bonus: WLAN testen

```bash
iw dev wlp2s0 link                # Verbindung prüfen
ip route | grep default           # Aktives Interface
ping -I wlp2s0 8.8.8.8            # Ping über WLAN
```

---
Klaro, hier ist der **Sicherheits-Abschnitt** für deine README – einfach hinten dranklatschen:

---

````markdown
## 🔒 5. Grundschutz & Sicherheit

### ClamAV (Virenscanner + Daemon)
```bash
sudo apt install clamav clamav-daemon -y
sudo systemctl enable clamav-freshclam --now
sudo freshclam
````

### chkrootkit (Rootkit-Scanner)

```bash
sudo apt install chkrootkit -y
sudo chkrootkit
```

### rkhunter (Rootkit Hunter)

```bash
sudo apt install rkhunter -y
sudo rkhunter --update
sudo rkhunter --propupd
sudo rkhunter --check
```

### fail2ban (Schutz vor Brute-Force)

```bash
sudo apt install fail2ban -y
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```

**Tipp:** Logins per SSH absichern mit `[sshd]`-Block in `jail.local`.

### Check & Status

```bash
sudo systemctl status clamav-daemon
sudo fail2ban-client status
sudo chkrootkit
sudo rkhunter --check
```




