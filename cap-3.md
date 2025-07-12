# 🔧 Cap-3: Performance & Resilience Test Suite for ThinkPad X201 (Tor Edition)

Welcome to Cap-3 — the testing chapter of your Tor-based infrastructure on legacy systems.  
This guide provides **non-intrusive, install-free** performance and security-related tests, designed for **paranoid setups** like your trusted ThinkPad X201.

> 💡 All tests avoid unnecessary installations. Ideal for hardened or air-gapped devices.

---

## 📦 Contents

- [CPU: Hash Testing](#cpu-hash-testing)
- [Disk: Read/Write Benchmark](#disk-readwrite-benchmark)
- [Thermals: Live Temp Monitor](#thermals-live-temp-monitor)
- [SMART: Disk Health (Optional)](#smart-disk-health-optional)
- [Network: Ping & DNS](#network-ping--dns)
- [Tor: Config Validation](#tor-config-validation)
- [RAM & Load: Quick Checks](#ram--load-quick-checks)
- [Systemd Boot Analysis](#systemd-boot-analysis)
- [Credits](#credits)

---

## 🧠 CPU: Hash Testing

```bash
openssl speed sha256 for our x201
````
<details>
<summary>openssl speed sha256 Output</summary>

    Doing sha256 for 3s on 16 size blocks: 5374056 sha256's in 3.00s
    Doing sha256 for 3s on 64 size blocks: 3564538 sha256's in 2.99s
    Doing sha256 for 3s on 256 size blocks: 1729758 sha256's in 2.99s
    Doing sha256 for 3s on 1024 size blocks: 552553 sha256's in 3.00s
    Doing sha256 for 3s on 8192 size blocks: 77320 sha256's in 3.00s
    Doing sha256 for 3s on 16384 size blocks: 39082 sha256's in 2.99s
    version: 3.0.13
    built on: Wed Feb  5 13:17:43 2025 UTC
    options: bn(64,64)
    compiler: gcc -fPIC -pthread -m64 -Wa,--noexecstack -Wall -fzero-call-used-regs=used-gpr -DOPENSSL_TLS_SECURITY_LEVEL=2 -Wa,--noexecstack -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/build/openssl-7xongr/openssl-3.0.13=. -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/build/openssl-7xongr/openssl-3.0.13=/usr/src/openssl-3.0.13 -DOPENSSL_USE_NODELETE -DL_ENDIAN -DOPENSSL_PIC -DOPENSSL_BUILDING_OPENSSL -DNDEBUG -Wdate-time -D_FORTIFY_SOURCE=3
    CPUINFO: OPENSSL_ia32cap=0x29ae3ffffebffff:0x0
    The 'numbers' are in 1000s of bytes per second processed.
    type             16 bytes     64 bytes    256 bytes   1024 bytes   8192 bytes   16384 bytes
    sha256           28661.63k    76297.80k   148099.68k   188604.76k   211135.15k   214153.67k

</details>


Optional (single run hash):

```bash
time openssl dgst -sha256 /bin/bash
```

<details>
<summary>openssl single run hash speed Output</summary>
    
    SHA2-256(/bin/bash)= bc594xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
    real    0m0,026s
    user    0m0,022s
    sys     0m0,004s

</details>

---

## 💾 Disk: Read/Write Benchmark

### Write Speed

```bash
dd if=/dev/zero of=~/testfile bs=1M count=512 status=progress
```

<details>
<summary>Disk write speed Output/summary>

    512+0 records in
    512+0 records out
    536870912 bytes (537 MB, 512 MiB) copied, 0,466591 s, 1,2 GB/s

</details>

### Read Speed

```bash
dd if=~/testfile of=/dev/null bs=1M status=progress
```

### Clean-up

```bash
rm ~/testfile
```

---

## 🌡️ Thermals: Live Temp Monitor

```bash
watch -n 2 cat /sys/class/thermal/thermal_zone*/temp
```

> Output in millidegrees (60000 = 60°C)

---

## 🔍 SMART: Disk Health (Optional)

```bash
sudo smartctl -a /dev/sda
```

If not installed:

```bash
sudo apt install smartmontools
```

---

## 🌍 Network: Ping & DNS

### Ping Tor Project

```bash
ping -c 5 check.torproject.org
```

### Local DNS Resolution

```bash
dig check.torproject.org @127.0.0.1
```

---

## 🛡️ Tor: Config Validation

```bash
sudo -u debian-tor /usr/sbin/tor -f /etc/tor/instances/hidden_service_1/torrc --verify-config
```

> No Tor instance is started. Just checks config sanity.

---

## 🧮 RAM & Load: Quick Checks

```bash
grep MemTotal /proc/meminfo
free -m
cat /proc/loadavg
```

---

## ⏱️ Systemd Boot Analysis

```bash
systemd-analyze blame
```

> Great for identifying startup bottlenecks on older hardware.

## 🔌 Hardware Health Checks

### USB & Ports
```bash
# USB devices und deren Power draw
lsusb -t
cat /sys/kernel/debug/usb/devices | grep -E "(Product|Manufacturer|MaxPower)"
```

### Memory Stress Test (ohne Installation)
```bash
# RAM mit /dev/urandom füllen (vorsichtig!)
stress-ng --vm 1 --vm-bytes 75% --timeout 30s
# Falls stress-ng fehlt:
dd if=/dev/urandom of=/dev/null bs=1M count=1024 & 
```

## ⚡ Power & Thermals Erweitert

### CPU Frequency Monitoring
```bash
# CPU scaling governor check
cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
# Real-time CPU freq
watch -n 1 "cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_cur_freq"
```

### Fan Control (X201 specific)
```bash
# ThinkPad fan status
cat /proc/acpi/ibm/fan
# Thermal throttling check
dmesg | grep -i "thermal\|throttl"
```

## 🔧 I/O & Storage Deep Dive

### Disk I/O unter Last
```bash
# Random I/O test
dd if=/dev/urandom of=~/random_test bs=4K count=10000 oflag=direct
# Latency test
time sync
```

### File System Health
```bash
# Inode usage
df -i
# Mount options check
mount | grep -E "(ext4|ntfs|fat)"
```

## 🖥️ Display & Graphics (für X201)
```bash
# GPU info (Intel GMA)
lspci | grep VGA
cat /sys/class/drm/card0/device/power_state
# Screen brightness range
cat /sys/class/backlight/*/max_brightness
```

## 🔊 Audio Hardware
```bash
# Audio devices
cat /proc/asound/cards
# Volume levels
amixer sget Master
```

## ⌨️ Input Devices
```bash
# Keyboard/Trackpad events
cat /proc/bus/input/devices | grep -A 5 -B 5 "keyboard\|mouse"
# X201 TrackPoint check
xinput list | grep -i track
```


---

## 🙌 Credits

This diagnostic chapter was co-developed with ❤️ by
**S. Volkan Kücükbudak (aka Batman)** and
**ChatGPT & Claude (a slightly overclocked T-Rex)** —
inspired by a stubborn ThinkPad X201 that refuses to die.

**Mission:** Build tools for the good, open-source for the right reasons, and make the world just a bit harder to control by the wrong people.

---

> "Some systems whisper when others scream. This one hisses like a survivor." — *Cap-3 Quote*
