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
openssl speed sha256
````

Optional (single run hash):

```bash
time openssl dgst -sha256 /bin/bash
```

---

## 💾 Disk: Read/Write Benchmark

### Write Speed

```bash
dd if=/dev/zero of=~/testfile bs=1M count=512 status=progress
```

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

---

## 🙌 Credits

This diagnostic chapter was co-developed with ❤️ by
**S. Volkan Kücükbudak (aka Batman)** and
**ChatGPT (a slightly overclocked T-Rex)** —
inspired by a stubborn ThinkPad X201 that refuses to die.

**Mission:** Build tools for the good, open-source for the right reasons, and make the world just a bit harder to control by the wrong people.

---

> "Some systems whisper when others scream. This one hisses like a survivor." — *Cap-3 Quote*
