# 🔍 Home Network Security Audit

> **Passive reconnaissance lab exercise** — personal home network · No configurations changed · May 2026

---

## Overview

A self-directed network security audit performed on my home LAN using Nmap. The goal was to identify exposed services, assess risk levels per device, and document findings the way a real security report would — without exploiting or modifying any device.

**Scope:** `10.0.0.0/24`  
**Tool:** Nmap 7.94SVN  
**Method:** Passive scan — service/version detection + OS fingerprinting  
**Hosts discovered:** 10 of 256  
**Scan time:** 52 seconds  

---

## Summary

| Severity | Count |
|----------|-------|
| 🔴 Critical | 1 |
| 🟡 Medium | 3 |
| 🟢 Low | 2 |
| 🔵 Informational | 4 |

---

## Device Inventory

| IP | Device | Open Ports | Risk |
|----|--------|------------|------|
| `10.0.0.1` | Xfinity Broadband Router (Technicolor) | 53, 80, 443 | 🟡 Medium |
| `10.0.0.5` | IP Camera #1 | 443, 554 (RTSP) | 🟡 Medium |
| `10.0.0.90` | IP Camera #2 | 5000 (RTSP) | 🟡 Medium |
| `10.0.0.220` | Windows PC — MySQL Server (Intel NIC) | 3306 | 🔴 Critical |
| `10.0.0.81` | Xiaomi Android Phone | None | 🟢 Secure |
| `10.0.0.71` | Unknown device | 49152 | 🔵 Info |
| `10.0.0.58` | Unknown (AzureWave NIC) | None visible | 🔵 Info |
| `10.0.0.120` | Unknown device | None visible | 🔵 Info |
| `10.0.0.144` | Unknown (Intel NIC) | None visible | 🔵 Info |
| `10.0.0.10` | Local host | None | 🟢 Low |

---

## Findings

### 🔴 F-01 — MySQL exposed on LAN (Critical)

**Host:** `10.0.0.220`  
**Port:** `3306/tcp open`  
**Detail:** MySQL is reachable from any device on the LAN. Nmap flagged the response as `unauthorized`, meaning the service responds to connections before any application-level auth. This is the highest-risk finding — any device on the same network can attempt to connect directly to the database.

**Fix:** Bind MySQL to `127.0.0.1` only, or add a firewall rule blocking port 3306 from LAN access.

```bash
# In /etc/mysql/mysql.conf.d/mysqld.cnf
bind-address = 127.0.0.1
```

---

### 🟡 F-02 — Unauthenticated RTSP camera streams (Medium)

**Hosts:** `10.0.0.5` (port 554), `10.0.0.90` (port 5000)  
**Detail:** Both IP cameras expose RTSP streams on the LAN with no confirmed authentication. RTSP is a real-time video streaming protocol — an unauthenticated stream means anyone on the network can potentially view the live feed.

**Fix:** Enable RTSP authentication in each camera's admin panel. Use a strong, unique password. Consider placing cameras on an isolated IoT VLAN.

---

### 🟡 F-03 — dnsmasq 2.83 on router (Medium)

**Host:** `10.0.0.1`  
**Port:** `53/tcp open`  
**Detail:** The router runs dnsmasq version 2.83, which has publicly documented CVEs. While filtered ports (SSH :22, Telnet :23) indicate reasonable baseline hardening, the DNS service version should be patched.

**Fix:** Check for Xfinity/Technicolor firmware updates. The router admin panel is accessible at `http://10.0.0.1`.

---

### 🔵 F-04 — Unidentified hosts (Informational)

**Hosts:** `10.0.0.58`, `10.0.0.71`, `10.0.0.120`, `10.0.0.144`  
**Detail:** Four hosts responded to ping but most ports were filtered or closed in the top-100 scan. One host (`.71`) exposed port `49152/tcp` (tcpwrapped — common on Apple/Windows UPnP). Full identification requires a deeper scan.

**Next step:**
```bash
nmap -p- -T4 10.0.0.58
nmap -p- -T4 10.0.0.71
```

---

## What Was Secure

- **Xiaomi phone (`10.0.0.81`)** — All 100 scanned ports closed. Device firewall functioning correctly.
- **Router filtered ports** — SSH (22) and Telnet (23) are filtered, not open. Good baseline hardening.
- **Router HTTP headers** — Security headers present: `X-Frame-Options`, `X-XSS-Protection`, `Content-Security-Policy`, `Strict-Transport-Security`.

---

## Recommendations

| Priority | Action |
|----------|--------|
| 🔴 Immediate | Bind MySQL to localhost — block port 3306 from LAN |
| 🟡 High | Enable authentication on both RTSP camera streams |
| 🟡 High | Update router firmware to patch dnsmasq CVEs |
| 🟢 Medium | Place IoT cameras on a separate VLAN |
| 🔵 Low | Run full port scan on 4 unidentified devices |

---

## Methodology

```
Phase 1 — Host discovery      nmap -sn 10.0.0.0/24
Phase 2 — Port + service scan nmap -sV -O --top-ports 100 10.0.0.0/24
Phase 3 — Analysis            Manual review of service fingerprints and CVE lookup
Phase 4 — Report              Risk rating per device, remediation steps
```

No exploitation was performed. No device configurations were modified. This is a passive reconnaissance exercise for educational purposes.

---

## Skills Demonstrated

- Network reconnaissance with Nmap
- Service and OS fingerprinting
- CVE awareness and risk rating
- Security report writing
- Remediation planning

---

## Disclaimer

> This audit was performed on my own home network for educational purposes as part of a personal security lab. All devices on this network are owned by me. No unauthorized access was attempted or achieved.

---


