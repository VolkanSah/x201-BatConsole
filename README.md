# x201 – BatConsole Setup & Tipps

Was macht man mit nem alten Lenovo X201?
Hier die besten Tricks, damit die Kiste zuverlässig, kühl und performant läuft – auch als Server mit WLAN.

---

## 1. Systemoptimierung & Kühlung

### tlp installieren

Akku & CPU effizient managen (auch im Netzbetrieb):

```bash
sudo apt install tlp tlp-rdw
sudo systemctl enable tlp --now
```

---

### CPU-Governor festlegen

Peaks bei unnötiger Last drosseln:

```bash
sudo apt install cpufrequtils
echo 'GOVERNOR="powersave"' | sudo tee /etc/default/cpufrequtils
sudo systemctl restart cpufrequtils
```

**Tipp:** Alternativ konservativ für bessere Balance:

```bash
sudo cpufreq-set -g conservative
```

Oder dauerhaft ändern:

```bash
echo 'GOVERNOR="conservative"' | sudo tee /etc/default/cpufrequtils
sudo systemctl restart cpufrequtils
```

---

### Temperatur & Throttling live checken

```bash
watch -n 1 "sensors && cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor"
```

Oder CPU-Frequenz beobachten:

```bash
watch -n 1 "grep 'MHz' /proc/cpuinfo"
```

---

## 2. BIOS-Einstellungen (sofern zugänglich)

* Hyperthreading **an** (aus = weniger Hit) ✅
* Intel Turbo Boost **aus** (nur wenn’s zu heiß wird) ❌
* Lüftersteuerung auf „Performance“ stellen, falls möglich

---

## 3. Lüfter & Stress-Test (Nice-to-have)

```bash
sudo apt install fancontrol pwmconfig stress s-tui
sudo pwmconfig        # Lüfterdrehzahl anpassen (vorsichtig bei Laptops)
stress --cpu 4        # CPU-Stresstest (je nach Kernanzahl)
s-tui                 # Terminal CPU Temperatur & Last Monitor
```

---

## 4. WLAN Setup (erstes Mal auf Server)

Datei: `/etc/wpa_supplicant/wpa_supplicant.conf` anlegen oder bearbeiten:

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

Nach Änderungen WLAN neu starten oder System rebooten, damit die Verbindung automatisch klappt.

network:
  version: 2
  renderer: networkd
  wifis:
    wlp2s0:
      dhcp4: true
      access-points:
        "DeinWLANName":
          password: "DeinPasswort"
Fix in 2 Sekunden:

sudo chmod 600 /etc/netplan/01-netcfg.yaml

Danach erneut:

sudo netplan apply

