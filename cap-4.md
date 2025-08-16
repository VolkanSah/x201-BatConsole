# 🛡️ Cap-4: Security Audit Suite for Tor Edition Systems

Welcome to Cap-4 — the comprehensive security audit chapter for your Tor-based infrastructure.  
This guide provides **paranoid-level security checks** designed for legacy systems like the ThinkPad X201 and modern servers alike.

> 🔒 **Philosophy:** Trust nothing, verify everything. These tests assume you're running a hardened system and want to keep it that way.

---

## 📦 Contents

- [System Hardening Verification](#-system-hardening-verification)
- [Tor Security Audit](#-tor-security-audit)
- [Network Security Analysis](#-network-security-analysis)
- [File System Security](#-file-system-security)
- [Process & Service Audit](#p-rocess--service-audit)
- [Memory & Kernel Security](#-memory--kernel-security)
- [User & Permission Audit](#-user--permission-audit)
- [Forensic Footprint Analysis](#-forensic-footprint-analysis)
- [Anonymity Leak Detection](#-anonymity-leak-detection)
- [Emergency Lockdown Checks](#-emergency-lockdown-checks)
- [Credits](#-credits)

---

## 🔐 System Hardening Verification

### Kernel Security Parameters
```bash
# ASLR (should be 2)
cat /proc/sys/kernel/randomize_va_space

# Kernel pointer restriction (should be 1 or 2)
cat /proc/sys/kernel/kptr_restrict

# Dmesg restriction (should be 1)
cat /proc/sys/kernel/dmesg_restrict

# Core dump restriction
cat /proc/sys/fs/suid_dumpable

# Kernel module loading restriction
cat /proc/sys/kernel/modules_disabled
```

### Boot Security
```bash
# Check if secure boot is enabled
mokutil --sb-state 2>/dev/null || echo "mokutil not found"

# Verify no insecure kernel parameters
cat /proc/cmdline | grep -E "(init=/bin/sh|single|emergency|rescue)"

# Check for unsigned kernel modules
find /lib/modules/$(uname -r) -name "*.ko" -exec modinfo {} \; | grep -c "signature"
```

### File System Security
```bash
# Check mount options for security
mount | grep -E "(nodev|nosuid|noexec)" --color=always

# Verify no world-writable files in critical paths
find /etc /bin /sbin /usr/bin /usr/sbin -perm -002 -type f 2>/dev/null

# Check for SUID/SGID files
find / -perm -4000 -o -perm -2000 2>/dev/null | head -20
```

---

## 🧅 Tor Security Audit

### Tor Configuration Security
```bash
# Verify Tor is running with correct user
ps aux | grep tor | grep -v grep

# Check Tor configuration for security issues
sudo grep -E "(Log|DataDirectory|HiddenServiceDir)" /etc/tor/torrc

# Verify no clearnet DNS leaks in Tor config
sudo grep -E "(DNS|Resolve)" /etc/tor/torrc

# Check hidden service permissions
sudo find /var/lib/tor -type f -exec ls -la {} \; 2>/dev/null
```

### Tor Circuit Analysis
```bash
# Check if Control Port is properly secured
sudo netstat -tlnp | grep :9051

# Verify Tor process isolation
sudo ls -la /proc/$(pgrep tor)/fd/ 2>/dev/null | head -10

# Check for Tor log security
sudo find /var/log -name "*tor*" -exec ls -la {} \; 2>/dev/null
```

### Hidden Service Security
```bash
# Check hidden service key permissions (should be 600)
sudo find /var/lib/tor/hidden_service* -name "*.key" -exec ls -la {} \; 2>/dev/null

# Verify no backup files in hidden service dirs
sudo find /var/lib/tor/hidden_service* -name "*~" -o -name "*.bak" 2>/dev/null

# Check hidden service hostname files
sudo find /var/lib/tor/hidden_service* -name "hostname" -exec cat {} \; 2>/dev/null
```

---

## 🌐 Network Security Analysis

### Network Interfaces & Routing
```bash
# Check for IPv6 (should be disabled in paranoid setups)
ip -6 addr show | grep -v "::1"

# Verify no unexpected network interfaces
ip link show | grep -E "(wlan|wifi|bluetooth|bt)"

# Check routing table for anomalies
ip route show table all
```

### Port & Service Analysis
```bash
# Check for open ports (should be minimal)
ss -tuln | grep -E ":(22|80|443|9050|9051|25|587|993|995)"

# Verify no suspicious listening services
sudo netstat -tlnp | grep -v "127.0.0.1\|::1"

# Check for established connections
ss -tuln | grep ESTAB
```

### Firewall Status
```bash
# Check iptables rules
sudo iptables -L -n -v

# Verify UFW status (if used)
sudo ufw status verbose 2>/dev/null || echo "UFW not installed"

# Check for nftables rules
sudo nft list ruleset 2>/dev/null || echo "nftables not active"
```

---

## 📂 File System Security

### Critical File Permissions
```bash
# Check /etc/passwd and /etc/shadow permissions
ls -la /etc/passwd /etc/shadow /etc/group

# Verify SSH key permissions
find ~/.ssh /etc/ssh -type f -exec ls -la {} \; 2>/dev/null

# Check for world-readable private keys
find / -name "*.key" -o -name "id_*" 2>/dev/null | xargs ls -la 2>/dev/null
```

### Sensitive File Discovery
```bash
# Look for password files
find /home /root -name "*password*" -o -name "*passwd*" 2>/dev/null

# Check for backup files with sensitive data
find / -name "*.bak" -o -name "*~" -o -name "*.old" 2>/dev/null | head -10

# Search for cryptocurrency wallets
find /home -name "wallet.dat" -o -name "*.wallet" 2>/dev/null
```

### Temporary File Security
```bash
# Check /tmp permissions and mount options
ls -lad /tmp /var/tmp
mount | grep -E "(tmp|shm)"

# Look for sensitive data in temp directories
find /tmp /var/tmp -type f -name "*" 2>/dev/null | head -10
```

---

## 🔍 Process & Service Audit

### Running Process Analysis
```bash
# Check for suspicious processes
ps aux | grep -E "(ssh|vnc|rdp|teamviewer|chrome|firefox)" | grep -v grep

# Verify no processes running as root unnecessarily
ps aux | awk '$1=="root" {print $1,$2,$11}' | grep -v -E "(kernel|init|kthread)"

# Check for processes with unusual network activity
sudo lsof -i -n | grep -v "127.0.0.1\|::1"
```

### Service Security
```bash
# Check enabled systemd services
systemctl list-unit-files --type=service --state=enabled | grep -v "systemd"

# Verify no unwanted services are running
systemctl --type=service --state=running | grep -E "(ssh|apache|nginx|ftp|telnet|rsh)"

# Check for failed services (potential security issues)
systemctl --failed
```

---

## 🧠 Memory & Kernel Security

### Memory Protection
```bash
# Check for swap usage (should be minimal or none)
swapon --show
free -m | grep Swap

# Verify no core dumps
find /var/crash /tmp /var/tmp -name "core*" -o -name "*.core" 2>/dev/null

# Check for memory overcommit settings
cat /proc/sys/vm/overcommit_memory
```

### Kernel Module Security
```bash
# List loaded modules
lsmod | head -20

# Check for suspicious modules
lsmod | grep -E "(rootkit|backdoor|keylog)"

# Verify module signatures
grep -E "(module|signature)" /proc/sys/kernel/modules_disabled
```

---

## 👤 User & Permission Audit

### User Account Security
```bash
# Check for users with UID 0 (should only be root)
awk -F: '$3==0 {print $1}' /etc/passwd

# Verify no users with empty passwords
sudo awk -F: '$2=="" {print $1}' /etc/shadow

# Check for users with shell access
grep -E "/bin/(bash|sh|zsh|fish)" /etc/passwd
```

### Sudo & Permission Analysis
```bash
# Check sudo configuration
sudo visudo -c

# Verify no passwordless sudo
sudo grep -E "NOPASSWD" /etc/sudoers /etc/sudoers.d/* 2>/dev/null

# Check for unusual file capabilities
getcap -r / 2>/dev/null | head -10
```

---

## 🕵️ Forensic Footprint Analysis

### Login & Access History
```bash
# Check recent logins
last | head -15

# Verify no unusual login times
last | grep -E "(0[0-5]:|2[2-3]:)"

# Check for failed login attempts
sudo grep "Failed password" /var/log/auth.log | tail -10 2>/dev/null
```

### System Activity Traces
```bash
# Check command history size
echo "HISTSIZE: $HISTSIZE, HISTFILESIZE: $HISTFILESIZE"

# Look for sensitive commands in history
grep -E "(wget|curl|ssh|scp|rsync)" ~/.bash_history ~/.zsh_history 2>/dev/null | tail -5

# Check for log tampering
sudo find /var/log -name "*.log" -exec ls -la {} \; | grep -E "(Jan 01|1970)"
```

---

## 🎭 Anonymity Leak Detection

### Time & Timezone Security
```bash
# Check timezone (should be UTC for anonymity)
timedatectl status

# Verify NTP is not leaking info
timedatectl show-timesync 2>/dev/null || echo "systemd-timesyncd not active"

# Check for time skew
date && hwclock -r
```

### DNS & Network Leaks
```bash
# Check DNS configuration
cat /etc/resolv.conf

# Verify no IPv6 DNS leaks
dig AAAA google.com 2>/dev/null | grep -A 5 "ANSWER SECTION"

# Check for WebRTC leaks (if GUI available)
netstat -rn | grep -E "(default|0.0.0.0)"
```

### Browser & Application Fingerprinting
```bash
# Check for browser profiles
find /home -name ".mozilla" -o -name ".chrome" -o -name ".chromium" 2>/dev/null

# Verify no Java/Flash (fingerprinting vectors)
which java javac 2>/dev/null || echo "Java not found (good)"
find /usr -name "*flash*" 2>/dev/null | head -5
```

---

## 🚨 Emergency Lockdown Checks

### Quick Panic Verification
```bash
# Check if emergency scripts exist
ls -la /usr/local/bin/panic* /home/*/bin/panic* 2>/dev/null

# Verify disk encryption status
lsblk -f | grep -E "(crypto|crypt)"

# Check for emergency network kill switch
ip link show | grep -E "(wlan|wifi|eth)"
```

### Data Destruction Readiness
```bash
# Check for secure deletion tools
which shred wipe srm 2>/dev/null

# Verify no sensitive data in RAM dumps
sudo strings /proc/kcore | grep -E "(password|key|secret)" 2>/dev/null | head -3
```

---

## 📊 Security Score Summary

### Generate Security Report
```bash
# Create a quick security summary
echo "=== SECURITY AUDIT SUMMARY ===" > /tmp/security_report.txt
echo "Date: $(date)" >> /tmp/security_report.txt
echo "System: $(uname -a)" >> /tmp/security_report.txt
echo "Tor Status: $(systemctl is-active tor)" >> /tmp/security_report.txt
echo "Open Ports: $(ss -tuln | wc -l)" >> /tmp/security_report.txt
echo "Root Processes: $(ps aux | awk '$1=="root"' | wc -l)" >> /tmp/security_report.txt
echo "SUID Files: $(find / -perm -4000 2>/dev/null | wc -l)" >> /tmp/security_report.txt
cat /tmp/security_report.txt
```

---

## 🙌 Credits

This security audit framework was battle-tested with ❤️ by  
**S. Volkan Sah ** on legacy and modern systems alike.

**Mission:** Build paranoid-level security for the good, open-source for the right reasons, and make surveillance just a bit harder for the wrong people.

---

> "In security, paranoia is not a bug — it's a feature." — *Cap-4 Motto*

---

## 🔧 Quick Start Security Check

```bash
# One-liner for quick audit
echo "Quick Security Check:" && \
echo "ASLR: $(cat /proc/sys/kernel/randomize_va_space)" && \
echo "Swap: $(swapon --show | wc -l)" && \
echo "Tor: $(systemctl is-active tor)" && \
echo "IPv6: $(ip -6 addr show | grep -v ::1 | wc -l)" && \
echo "Open Ports: $(ss -tuln | grep -v 127.0.0.1 | wc -l)" && \
echo "Root Procs: $(ps aux | awk '$1=="root"' | wc -l)"
```
### Chapters

- [Cap-2: x201 – Web Server & Database Setup](cap-2.md)
- [Cap-3: Performance & Resilience Test (Tor Edition)](cap-3.md)
- [Cap-4: Security Audit Suite for Tor Edition Systems](cap-4.md)
- [Cap-5: Server Hardening](cap-5.md)
- [Why Ubuntu and not Debain?)](why-ubuntu.md)
- [Why Apache and not NGINX?](why-apache.md)


