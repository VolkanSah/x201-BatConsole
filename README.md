#  x201 – BatConsole Setup & Tuning Guide
##  What to do with an old Lenovo X201?

Turn it into a whisper-quiet, stable, Wi-Fi-enabled dev or home server — because heroes *don’t* let hardware rot in drawers.

> This repo isn’t just for Lenovo’s ancient but loyal X201 —
> It’s a basic, no-nonsense server setup you can apply to almost any hardware: old laptops, desktops, even that dusty machine in your basement.
>
> Prefer Debian? That’s fine too — the setup is nearly identical.
>
> Curious why we use Ubuntu instead of Debian?
> 👉 [Read more here](why-ubuntu.md)


# Table of Contents

- [1. System Optimization & Cooling](#️-1-system-optimization--cooling)
  - [TLP for Battery & CPU Optimization](#tlp-for-battery--cpu-optimization)
  - [Set CPU Governor](#set-cpu-governor)
    - [Recommended (Permanent)](#recommended-permanent)
  - [Monitor Temperatures](#monitor-temperatures)
    - [Display CPU Frequency](#display-cpu-frequency)
- [ 2. BIOS Tuning](#-2-bios-tuning)
- [3. Fan & Stress Testing (optional)](#3-fan--stress-testing-optional)
- [4. Wi-Fi Setup for Server Mode (2 options)](#-4-wi-fi-setup-for-server-mode-2-options)
  - [WPA Configuration](#wpa-configuration)
  - [Netplan Configuration](#netplan-configuration)
    - [Set Permissions & Apply](#-set-permissions--apply)
  - [Test Wi-Fi](#test-wi-fi)
- [5. Basic Protection & Security](#-5-basic-protection--security)
  - [Harden the System](#harden-the-system)
  - [ClamAV (Antivirus)](#clamav-antivirus)
  - [chkrootkit (Rootkit Detection)](#chkrootkit-rootkit-detection)
  - [rkhunter (Advanced Rootkit Detection)](#rkhunter-advanced-rootkit-detection)
  - [fail2ban (Brute-Force Protection)](#fail2ban-brute-force-protection)
    - [Example SSH Protection](#example-ssh-protection)
    - [Apply Config](#apply-config)
- [Final Step](#-final-step)
- [Backup & Restore Script](#-backup--restore-script)
  - [Features](#features)
  - [Usage](#usage)
  - [Setup](#setup)
- [Cap-2: x201 – Web Server & Database Setup](cap-2.md)
- [Cap-3: Performance & Resilience Test (Tor Edition)](cap-3.md)
- [Cap-4: Security Audit Suite for Tor Edition Systems](cap-3.md)
- [Why Ubuntu and not Debain?)](why-ubuntu.md)
- [Why Apache and not NGINX?](why-apache.md)
- [Support Note & Human Sanity Disclaimer™](#-support-note--human-sanity-disclaimer)
- [Support](#support)
- [License](#license)




## ⚙️ 1. System Optimization & Cooling

### TLP for Battery & CPU Optimization

```bash
sudo apt install tlp tlp-rdw
sudo systemctl enable tlp --now
```

###  Set CPU Governor

```bash
sudo apt install cpufrequtils
echo 'GOVERNOR="powersave"' | sudo tee /etc/default/cpufrequtils
sudo systemctl restart cpufrequtils
```

####  Recommended (Permanent)

```bash
echo 'GOVERNOR="conservative"' | sudo tee /etc/default/cpufrequtils
sudo systemctl restart cpufrequtils
```

###  Monitor Temperatures

```bash
sudo apt install lm-sensors
sudo sensors-detect
watch -n 1 "sensors && cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor"
```

####  Display CPU Frequency

```bash
watch -n 1 "grep 'MHz' /proc/cpuinfo"
```



## 🧬 2. BIOS Tuning

* ✅ Enable **Hyper-Threading**
* ❌ Disable **Intel Turbo Boost** (if overheating)
* 🔧 Set fan control to **Performance** (if available)



##  3. Fan & Stress Testing (optional)

```bash
sudo apt install fancontrol stress s-tui
```


## 📶 4. Wi-Fi Setup for Server Mode (2 options)

###  WPA Configuration (`/etc/wpa_supplicant/wpa_supplicant.conf`)

```conf
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=DE

network={
    ssid="YourNetworkName"
    psk="YourPassword"
    key_mgmt=WPA-PSK
}
```

###  Netplan Configuration (`/etc/netplan/01-netcfg.yaml`)

```yaml
network:
  version: 2
  renderer: networkd
  wifis:
    wlp2s0:
      dhcp4: true
      access-points:
        "YourNetworkName":
          password: "YourPassword"
```

#### 🔒 Set Permissions & Apply

```bash
sudo chmod 600 /etc/netplan/01-netcfg.yaml
sudo netplan apply
```

### Test Wi-Fi

```bash
iw dev wlp2s0 link                # Connection status
ip route | grep default           # Active interface
ping -I wlp2s0 8.8.8.8            # Ping test via Wi-Fi
```



## 🔒 5. Basic Protection & Security

###  Harden the System

```bash
# Disable Dash as /bin/sh (for compatibility)
sudo dpkg-reconfigure dash
# Choose “No” when asked for default system shell

# Disable AppArmor (if not needed)
sudo systemctl stop apparmor
sudo systemctl disable apparmor
sudo apt purge apparmor apparmor-utils -y
```
Update your etc/sysctl.conf like this [etc/sysctl.conf](etc/sysctl.conf)

### ClamAV (Antivirus)

```bash
sudo apt install clamav clamav-daemon -y
sudo systemctl enable clamav-freshclam --now
sudo freshclam
```

###  chkrootkit (Rootkit Detection)

```bash
sudo apt install chkrootkit -y
sudo chkrootkit
```

###  rkhunter (Advanced Rootkit Detection)

```bash
sudo apt install rkhunter -y
sudo rkhunter --update
sudo rkhunter --propupd
sudo rkhunter --check
```

###  fail2ban (Brute-Force Protection)

```bash
sudo apt install fail2ban -y
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

#### Example SSH Protection (`/etc/fail2ban/jail.local`):

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

#### Apply Config

```bash
sudo systemctl restart fail2ban
```





## 🎉 Final Step

Your X201 is now optimized and secured. Reboot the system to apply all changes:

```bash
sudo reboot
```



## 💾 Backup & Restore Script

To quickly and securely back up your system after a fresh setup, use the [Backup & Restore Script](https://github.com/VolkanSah/Debian-System-Backup-and-Restore-Script/).

### Features

* Saves installed packages, config files, and optionally the full file system (excluding core system dirs)
* Stores backups in `/backup/YYYYMMDD_HHMMSS` automatically
* Logs every step into the backup folder
* Restores packages, configs, and optionally full system structure

### Usage

**Create a backup:**

```bash
sudo /batscripts/backup.sh backup
```

For full system backup:

```bash
sudo /batscripts/backup.sh backup full
```

**Restore a backup:**

```bash
sudo /batscripts/backup.sh restore /backup/20250709_191251
```

Full restore including files:

```bash
sudo /batscripts/backup.sh restore /backup/20250709_191251 full
```

### Setup

* Place the script under e.g. `/batscripts/backup.sh`
* Grant execution: `sudo chmod +x /batscripts/backup.sh`
* Optional: Add an alias like `batbackup`

### Pro Tip

Use a cronjob for automatic backups and stay safe at all times.





---

## 🤖 Support Note & Human Sanity Disclaimer™

> This project was born not just from curiosity, but mostly because I was too lazy to remember every damn package and config flag.
> So I asked some AI buddies – ChatGPT, Deepseek, and a few others. They tried hard... but mostly just repeated the same sanitized tech-manual fluff.
> In the end, it always takes a stubborn human with a **Thank Pad** (yes, *ThinkPad*) to fix the chaos and make things actually work™.
>
> This whole setup was handcrafted with caffeine, rage against broken tutorials, deep system logs, and a little help from not-so-evil AI.
>
> ✨ **AI isn’t evil. But humans can be.** Let’s use the machine to build, not to break.
> 
> ⭐️ If this project helped you, drop a star.
> 
> 🥖 If you’re rich: sponsor me.
> 
> 🫡 If you’re broke too: respect – now go fix your own ThankPad™.

## Support

If this helped you:

* ⭐ the repo
* Share it
* Visit [Volkan Sah](https://github.com/volkansah)
* [Support via GitHub Sponsors](https://github.com/sponsors/volkansah)

---

## License

Privat License for Freedom — see LICENSE file.

---

**Credits:** 
- Mr.Chess alias Volan Sah
- Readme.md Powered by Batman’s grind and ChatGPT wizardry. 🦇🔥
