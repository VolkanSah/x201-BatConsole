# 🐧 Ubuntu vs Debian für den Lenovo X201 Server

**Empfehlung für dieses Projekt:**  
✅ **Ubuntu LTS** (24.04) bietet die beste Balance aus Treiber-Support und Stabilität für den X201.

---

## 🔍 Vergleichstabelle

| Feature               | Ubuntu LTS               | Debian Stable            |
|-----------------------|--------------------------|--------------------------|
| **Treiber**           | ✔️ Vollständige Firmware-Pakete | 🔧 `non-free`-Repo nötig |
| **WLAN-Support**      | Intel-Chips sofort ready | Oft manuelle Installation |
| **Power Management**  | TLP/thermald optimiert   | Grundkonfiguration       |
| **ARM-Builds**        | Offizielle RPi-Images    | Generische ARM-Pakete    |
| **Update-Zyklus**     | Alle 2 Jahre (5 Jahre Support) | Konservativere Updates |
| **Community-Support** | Große Laptop-Community   | Server-Fokus            |

---

## 🏆 Ubuntu-Vorteile für den X201

### 1. Out-of-the-Box Funktionalität
```bash
# Beispiel: WLAN-Treiber sind direkt da
lspci -k | grep -A 3 -i "network"
# Intel Centrino Advanced-N 6200 wird sofort erkannt
```

### 2. Bessere Power Management Tools
```bash
# Vorkonfiguriertes TLP:
sudo tlp-stat -b
# Angepasste thermald-Einstellungen für Laptops
```

### 3. Hardware-Enhancements
```bash
# Neue Mesa-Treiber via PPA:
sudo add-apt-repository ppa:kisak/kisak-mesa
sudo apt upgrade
```

---

## 🛠️ Wann Debian die bessere Wahl ist

### Ideal für:
- **Minimal-Setups** (kein Snapd, weniger Hintergrunddienste)
- **Ressourcenlimitierung** (Debian verbraucht ~100MB weniger RAM)
- **Langzeit-Stabilität** (Keine unerwarteten Major-Updates)

### Beispiel-Installation:
```bash
# Für Debian + WLAN-Support:
sudo apt install firmware-iwlwifi wireless-tools
```

---

## 🔄 Hybrid-Lösung: Debian mit Backports
```bash
# /etc/apt/sources.list:
deb http://deb.debian.org/debian bookworm-backports main contrib non-free
```
**Vorteile:**
- Debian-Stabilität + aktuellere Treiber
- Manuelle Kontrolle über Updates

---

## 🦇 X201-spezifische Empfehlung
```diff
+ Ubuntu 24.04 LTS
- Für maximale Kompatibilität mit:
  - Intel HD Graphics
  - SD-Kartenleser
  - ThinkPad-Sonderfunktionen (Hotkeys, etc.)
```

> **Tipp:** Für Server-Betrieb Ubuntu **minimal installieren** ohne GUI:  
> `sudo apt purge ubuntu-desktop && sudo apt autoremove`

---

## 📚 Weiterführende Links
- [Ubuntu LTS Release Notes](https://wiki.ubuntu.com/LTS)
- [Debian Hardware Compatibility List](https://wiki.debian.org/Hardware)
- [ThinkPad Optimizations Guide](https://github.com/thinkpad-guide)
