# 🖥️ x201 – BatConsole Setup & Tuning Guide
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

1. [System Optimization & Cooling](#-1-system-optimization--cooling)  
   1.1 [TLP for Battery & CPU Optimization](#tlp-for-battery--cpu-optimization)  
   1.2 [Set CPU Governor](#set-cpu-governor)  
   1.3 [Recommended (Permanent)](#recommended-permanent)  
   1.4 [Monitor Temperatures](#monitor-temperatures)  
   1.5 [Display CPU Frequency](#display-cpu-frequency)

2. [BIOS Tuning](#-2-bios-tuning)  
   2.1 [Enable Hyper-Threading](#enable-hyper-threading)  
   2.2 [Disable Intel Turbo Boost](#disable-intel-turbo-boost)  
   2.3 [Set Fan Control to Performance](#set-fan-control-to-performance)

3. [Fan & Stress Testing (optional)](#-3-fan--stress-testing-optional)

4. [Wi-Fi Setup for Server Mode (2 options)](#-4-wi-fi-setup-for-server-mode-2-options)  
   4.1 [WPA Configuration (`/etc/wpa_supplicant/wpa_supplicant.conf`)](#wpa-configuration-etcwpa_supplicantwpa_supplicantconf)  
   4.2 [Netplan Configuration (`/etc/netplan/01-netcfg.yaml`)](#netplan-configuration-etcnetplan01-netcfgyaml)  
   4.3 [Set Permissions & Apply](#set-permissions--apply)  
   4.4 [Test Wi-Fi](#test-wi-fi)

5. [Basic Protection & Security](#-5-basic-protection--security)  
   5.1 [Harden the System](#harden-the-system)  
   5.2 [ClamAV (Antivirus)](#clamav-antivirus)  
   5.3 [chkrootkit (Rootkit Detection)](#chkrootkit-rootkit-detection)  
   5.4 [rkhunter (Advanced Rootkit Detection)](#rkhunter-advanced-rootkit-detection)  
   5.5 [fail2ban (Brute-Force Protection)](#fail2ban-brute-force-protection)  
   5.6 [Example SSH Protection (`/etc/fail2ban/jail.local`)](#example-ssh-protection-etcfail2banjaillocal)  
   5.7 [Apply Config](#apply-config)

6. [Final Step](#-final-step)

7. [Backup & Restore Script](#-backup--restore-script)  
   7.1 [Features](#features)  
   7.2 [Usage](#usage)  
      7.2.1 [Create a backup](#create-a-backup)  
      7.2.2 [Full system backup](#full-system-backup)  
      7.2.3 [Restore a backup](#restore-a-backup)  
      7.2.4 [Full restore including files](#full-restore-including-files)  
   7.3 [Setup](#setup)  
   7.4 [Pro Tip](#pro-tip)

8. [Further Links](#-further-links)  
   8.1 [Cap-2: x201 – Web Server & Database Setup](#cap-2-x201--web-server--database-setup)  
   8.2 [Cap-3: Performance & Resilience Test (Tor Edition)](#cap-3-performance--resilience-test-tor-edition)  
   8.3 [Ubuntu vs Debian for a System like Lenovo X201 (Server)](#ubuntu-vs-debian-for-a-system-like-lenovo-x201-server)

9. [Support Note & Human Sanity Disclaimer™](#-support-note--human-sanity-disclaimer)

10. [Support](#-support)  
    10.1 [Star the repo](#star-the-repo)  
    10.2 [Share it](#share-it)  
    10.3 [Visit Volkan Sah](#visit-volkan-sah)  
    10.4 [Support via GitHub Sponsors](#support-via-github-sponsors)

11. [License](#-license)

12. [Credits](#-credits)




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



##### **For more on system tuning, BIOS settings, fan control, and other hardware specifics, check out:**

* [Cap-2: x201 – Web Server & Database Setup](cap-2.md)
* [Cap-3: Performance & Resilience Test (Tor Edition)](cap-3.md)
* [Ubuntu vs Debian for a System like Lenovo X201 (Server)](why-ubuntu.md)

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

MIT License — see LICENSE file.

---

**Credits:** 
- Mr.Chess alias Volan Sah
- Readme.md Powered by Batman’s grind and ChatGPT wizardry. 🦇🔥
