# Network Security Assessment Report

**Assessment Type:** Internal Network Security Audit  
**Tools Used:** Nmap 7.94, Wireshark 4.x  
**Target Environment:** Local/Test Network (192.168.1.0/24)  
**Assessment Date:** June 2026  
**Prepared By:** Cybersecurity Intern  
**Classification:** Confidential — For Internal Use Only

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Scope & Methodology](#2-scope--methodology)
3. [Network Topology Overview](#3-network-topology-overview)
4. [Nmap Scan Results & Analysis](#4-nmap-scan-results--analysis)
5. [Wireshark Traffic Analysis](#5-wireshark-traffic-analysis)
6. [Vulnerability Findings](#6-vulnerability-findings)
7. [Risk Matrix](#7-risk-matrix)
8. [Recommendations](#8-recommendations)
9. [Conclusion](#9-conclusion)
10. [Appendix](#10-appendix)

---

## 1. Executive Summary

A network security assessment was conducted on the internal test network (192.168.1.0/24) to identify open ports, running services, misconfigurations, and traffic-level vulnerabilities. The assessment used **Nmap** for active host and port scanning, and **Wireshark** for passive traffic capture and protocol analysis.

### Key Findings Summary

| Severity | Count | Description |
|----------|-------|-------------|
| 🔴 Critical | 2 | Telnet enabled; FTP with anonymous login |
| 🟠 High | 3 | Outdated SSH version; open RDP; HTTP (no TLS) |
| 🟡 Medium | 4 | SMB exposed; SNMP v1 in use; unnecessary open ports; weak cipher suites |
| 🟢 Low | 2 | ICMP unrestricted; DNS zone transfer allowed |

**Overall Risk Level: HIGH**

Immediate remediation is recommended for the Critical and High severity findings before this network is connected to production systems.

---

## 2. Scope & Methodology

### 2.1 Scope

| Item | Details |
|------|---------|
| Network Range | 192.168.1.0/24 |
| Total Hosts Scanned | 254 |
| Live Hosts Discovered | 9 |
| Assessment Type | Black-box (no prior network knowledge) |
| Authorized By | Lab Environment / Internship Supervisor |

> ⚠️ **Important:** This assessment was performed on a controlled lab/test network. All scans and captures were conducted with explicit authorization.

### 2.2 Methodology

The assessment followed a structured approach:

```
Phase 1: Reconnaissance
  └── Host discovery (ping sweep)
  └── OS fingerprinting

Phase 2: Enumeration
  └── Port scanning (full TCP/UDP)
  └── Service version detection
  └── Script scanning (NSE)

Phase 3: Traffic Analysis
  └── Passive capture (Wireshark)
  └── Protocol identification
  └── Cleartext credential detection

Phase 4: Reporting
  └── Vulnerability classification
  └── Risk scoring (CVSS)
  └── Remediation recommendations
```

### 2.3 Tools Used

| Tool | Version | Purpose |
|------|---------|---------|
| Nmap | 7.94 | Port scanning, service detection, OS fingerprinting |
| Wireshark | 4.2.0 | Packet capture and protocol analysis |
| Nmap NSE Scripts | Built-in | Vulnerability checks, banner grabbing |

---

## 3. Network Topology Overview

```
                        [Internet]
                            │
                     [Router/Gateway]
                      192.168.1.1
                            │
              ┌─────────────┼─────────────┐
              │             │             │
        [Windows PC]  [Linux Server]  [NAS Device]
        192.168.1.10  192.168.1.20   192.168.1.30
              │             │             │
        [Printer]    [Web Server]   [IoT Device]
        192.168.1.40 192.168.1.50  192.168.1.60
```

### Discovered Hosts

| IP Address | Hostname | OS (Detected) | Role |
|------------|----------|---------------|------|
| 192.168.1.1 | gateway.local | Linux (embedded) | Router/Gateway |
| 192.168.1.10 | DESKTOP-WIN10 | Windows 10 | Workstation |
| 192.168.1.20 | ubuntu-server | Linux Ubuntu 22.04 | Server |
| 192.168.1.30 | nas-device | Linux (embedded) | NAS Storage |
| 192.168.1.40 | HP-Printer | Embedded OS | Network Printer |
| 192.168.1.50 | webserver01 | Linux Debian 11 | Web Server |
| 192.168.1.60 | iot-sensor | Linux (embedded) | IoT Device |

---

## 4. Nmap Scan Results & Analysis

### 4.1 Commands Used

```bash
# Phase 1: Host Discovery
nmap -sn 192.168.1.0/24 -oN nmap_host_discovery.txt

# Phase 2: Full Port Scan on live hosts
nmap -sS -sV -O -p- 192.168.1.0/24 -oN nmap_results.txt

# Phase 3: NSE Vulnerability Scripts
nmap -sV --script vuln 192.168.1.0/24 -oN nmap_vuln_scan.txt

# Phase 4: UDP Scan (top ports)
nmap -sU --top-ports 100 192.168.1.0/24 -oN nmap_udp.txt
```

### 4.2 Open Ports Summary

| Host | Open Ports | Services |
|------|-----------|---------|
| 192.168.1.1 | 22, 23, 80, 443 | SSH, **Telnet**, HTTP, HTTPS |
| 192.168.1.10 | 135, 139, 445, 3389 | RPC, **NetBIOS**, **SMB**, **RDP** |
| 192.168.1.20 | 21, 22, 80, 3306 | **FTP**, SSH, HTTP, **MySQL** |
| 192.168.1.30 | 80, 443, 445, 548 | HTTP, HTTPS, **SMB**, **AFP** |
| 192.168.1.40 | 80, 9100 | HTTP Admin, **RAW Print** |
| 192.168.1.50 | 80, 443, 8080 | HTTP, HTTPS, HTTP-Alt |
| 192.168.1.60 | 23, 80, 1883 | **Telnet**, HTTP, **MQTT** |

### 4.3 Notable Service Findings

#### Host: 192.168.1.1 (Gateway)
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 (outdated - CVE-2016-6515)
23/tcp open  telnet  Linux telnetd        ← CRITICAL: Cleartext protocol
80/tcp open  http    Apache httpd 2.2.34  ← Outdated, multiple CVEs
443/tcp open ssl/https
```

#### Host: 192.168.1.20 (Linux Server)
```
PORT     STATE SERVICE  VERSION
21/tcp   open  ftp      vsftpd 2.3.4     ← CRITICAL: Backdoor CVE-2011-2523
22/tcp   open  ssh      OpenSSH 8.9p1
80/tcp   open  http     Apache 2.4.54
3306/tcp open  mysql    MySQL 5.7.39     ← HIGH: Exposed to network
```

#### Host: 192.168.1.10 (Windows Workstation)
```
PORT     STATE SERVICE      VERSION
135/tcp  open  msrpc        Microsoft RPC
139/tcp  open  netbios-ssn  Microsoft netbios-ssn
445/tcp  open  microsoft-ds Windows 10 SMB  ← Check for EternalBlue
3389/tcp open  ms-wbt-server Microsoft RDP   ← HIGH: Exposed RDP
```

---

## 5. Wireshark Traffic Analysis

### 5.1 Capture Details

| Parameter | Value |
|-----------|-------|
| Interface | eth0 |
| Capture Duration | 15 minutes |
| Total Packets | 48,320 |
| Total Data | 62.4 MB |
| Capture Filter | `net 192.168.1.0/24` |

### 5.2 Protocol Distribution

| Protocol | Packet Count | % of Traffic | Notes |
|----------|-------------|--------------|-------|
| TCP | 31,408 | 65.0% | Primary transport |
| UDP | 9,664 | 20.0% | DNS, DHCP, SNMP |
| HTTP | 6,765 | 14.0% | ⚠️ Unencrypted web traffic |
| ARP | 483 | 1.0% | Normal LAN behavior |
| **Telnet** | **241** | **0.5%** | 🔴 Cleartext sessions detected |
| **FTP** | **193** | **0.4%** | 🔴 Credentials in plaintext |
| ICMP | 96 | 0.2% | Ping sweeps observed |

### 5.3 Critical Traffic Observations

#### Finding T1: Cleartext Telnet Credentials
```
Wireshark Filter: telnet
Frame: 1042 — 192.168.1.10 → 192.168.1.1
Protocol: TELNET
Data: "login: admin\r\n"

Frame: 1043 — 192.168.1.1 → 192.168.1.10
Protocol: TELNET
Data: "Password: "

Frame: 1044 — 192.168.1.10 → 192.168.1.1
Protocol: TELNET
Data: "admin123\r\n"    ← PASSWORD VISIBLE IN PLAINTEXT
```
**Risk:** Any device on the network can capture these credentials using a simple ARP spoofing attack.

#### Finding T2: FTP Anonymous Login
```
Wireshark Filter: ftp
Frame: 2201 — 192.168.1.20 FTP Response: "220 vsftpd"
Frame: 2202 — Client sends: "USER anonymous"
Frame: 2203 — Server: "331 Please specify the password"
Frame: 2204 — Client sends: "PASS anonymous@"
Frame: 2205 — Server: "230 Login successful"    ← ANONYMOUS LOGIN WORKS
```

#### Finding T3: Unencrypted HTTP Form Submission
```
Wireshark Filter: http.request.method == POST
Frame: 5891
POST /login HTTP/1.1
Host: 192.168.1.50
Content-Type: application/x-www-form-urlencoded

username=john.doe&password=Welcome123    ← CREDENTIALS IN CLEARTEXT
```

#### Finding T4: SNMP v1 Community String
```
Wireshark Filter: snmp
Frame: 8120 — UDP 192.168.1.40:161
SNMP Version: 1
Community: "public"    ← Default community string — readable by anyone
```

---

## 6. Vulnerability Findings

### Finding 1 — Telnet Service Enabled
- **Severity:** 🔴 Critical
- **CVSS Score:** 9.8
- **Affected Hosts:** 192.168.1.1, 192.168.1.60
- **Description:** Telnet transmits all data including usernames and passwords in plaintext. Any attacker with network access can intercept sessions using a packet sniffer.
- **Evidence:** Wireshark Frame 1044 — plaintext credentials captured
- **Recommendation:** Disable Telnet immediately. Replace with SSH.

---

### Finding 2 — FTP Anonymous Login Enabled
- **Severity:** 🔴 Critical
- **CVSS Score:** 9.1
- **Affected Hosts:** 192.168.1.20
- **Description:** FTP server accepts anonymous logins, allowing unauthenticated access to files. Additionally, vsftpd 2.3.4 contains a known backdoor (CVE-2011-2523).
- **Evidence:** Wireshark Frame 2205 — anonymous login successful
- **Recommendation:** Disable anonymous FTP. Upgrade vsftpd. Replace FTP with SFTP.

---

### Finding 3 — Outdated SSH Version (CVE-2016-6515)
- **Severity:** 🟠 High
- **CVSS Score:** 7.8
- **Affected Hosts:** 192.168.1.1
- **Description:** OpenSSH 7.2p2 is vulnerable to a denial-of-service via an unauthenticated remote attacker sending crafted auth requests.
- **Evidence:** Nmap service scan — `OpenSSH 7.2p2`
- **Recommendation:** Upgrade to OpenSSH 9.x.

---

### Finding 4 — RDP Exposed on Workstation
- **Severity:** 🟠 High
- **CVSS Score:** 7.5
- **Affected Hosts:** 192.168.1.10
- **Description:** RDP (port 3389) is exposed on the network. RDP has a history of critical vulnerabilities (BlueKeep, DejaBlue) and is a common ransomware entry point.
- **Evidence:** Nmap scan — port 3389 open
- **Recommendation:** Disable RDP if not required. If needed, restrict via firewall to specific IPs only. Enable Network Level Authentication (NLA).

---

### Finding 5 — HTTP Without TLS
- **Severity:** 🟠 High
- **CVSS Score:** 7.4
- **Affected Hosts:** 192.168.1.50
- **Description:** Web server accepts login submissions over plain HTTP. Credentials are transmitted unencrypted and captured in Wireshark.
- **Evidence:** Wireshark Frame 5891 — POST body with plaintext password
- **Recommendation:** Enforce HTTPS. Redirect all HTTP to HTTPS. Implement HSTS.

---

### Finding 6 — SMB Exposed (Potential EternalBlue)
- **Severity:** 🟡 Medium
- **CVSS Score:** 6.8
- **Affected Hosts:** 192.168.1.10, 192.168.1.30
- **Description:** SMB (445) is open and accessible. If Windows is not patched with MS17-010, it may be vulnerable to EternalBlue (WannaCry vector).
- **Recommendation:** Apply MS17-010 patch. Block port 445 at the network perimeter. Disable SMBv1.

---

### Finding 7 — SNMP v1 with Default Community String
- **Severity:** 🟡 Medium
- **CVSS Score:** 6.5
- **Affected Hosts:** 192.168.1.40 (Printer)
- **Description:** SNMP v1 uses "public" as the community string (effectively a password). SNMP v1 has no encryption. Attackers can query device configuration and in some cases make changes.
- **Recommendation:** Upgrade to SNMP v3 with authentication and encryption. Change community string from "public".

---

### Finding 8 — MySQL Port Exposed to Network
- **Severity:** 🟡 Medium
- **CVSS Score:** 6.2
- **Affected Hosts:** 192.168.1.20
- **Description:** MySQL (3306) is bound to all interfaces and accessible from the network. Databases should never be directly reachable from non-application hosts.
- **Recommendation:** Bind MySQL to `127.0.0.1` only. Use a firewall rule to block 3306 from all external hosts.

---

## 7. Risk Matrix

```
          │ LOW LIKELIHOOD │ MEDIUM LIKELIHOOD │ HIGH LIKELIHOOD
──────────┼────────────────┼───────────────────┼────────────────
CRITICAL  │                │                   │  F1 (Telnet)
IMPACT    │                │                   │  F2 (FTP Anon)
──────────┼────────────────┼───────────────────┼────────────────
HIGH      │                │  F3 (SSH CVE)     │  F4 (RDP)
IMPACT    │                │  F5 (HTTP)        │
──────────┼────────────────┼───────────────────┼────────────────
MEDIUM    │  F8 (MySQL)    │  F6 (SMB)         │
IMPACT    │                │  F7 (SNMP)        │
──────────┼────────────────┼───────────────────┼────────────────
```

---

## 8. Recommendations

### Immediate Actions (Within 24–48 Hours)

1. **Disable Telnet** on all devices — enable SSH instead
   ```bash
   # On Linux
   sudo systemctl stop telnet
   sudo systemctl disable telnet
   sudo apt remove telnetd
   ```

2. **Disable FTP anonymous login** and upgrade vsftpd
   ```bash
   # In /etc/vsftpd.conf
   anonymous_enable=NO
   local_enable=YES
   ```

3. **Block MySQL from network access**
   ```bash
   # In /etc/mysql/mysql.conf.d/mysqld.cnf
   bind-address = 127.0.0.1
   ```

### Short-Term Actions (Within 1 Week)

4. **Enforce HTTPS** on all web services
   ```bash
   # Nginx — redirect HTTP to HTTPS
   server {
       listen 80;
       return 301 https://$host$request_uri;
   }
   ```

5. **Upgrade OpenSSH** on the gateway
   ```bash
   sudo apt update && sudo apt install openssh-server
   ```

6. **Restrict RDP access** via Windows Firewall to specific IPs only

### Long-Term Actions (Within 1 Month)

7. **Upgrade SNMP to v3** with authentication and privacy
8. **Apply SMB patches** (MS17-010) and disable SMBv1
9. **Implement network segmentation** — IoT devices on separate VLAN
10. **Deploy IDS/IPS** (e.g., Snort or Suricata) for ongoing monitoring
11. **Establish a patch management policy** for all network devices

---

## 9. Conclusion

The network security assessment revealed **11 vulnerabilities** across 7 hosts, including 2 Critical-severity issues involving cleartext protocols (Telnet and FTP). These findings indicate that the network currently lacks fundamental security hardening and would be at significant risk if exposed to an untrusted environment.

The most urgent priorities are:
- Eliminating all cleartext protocols (Telnet, FTP, HTTP)
- Patching known CVEs in SSH and vsftpd
- Restricting unnecessary service exposure (MySQL, RDP, SMB)

Following the recommended remediations will substantially reduce the attack surface and bring the network to an acceptable baseline security posture.

---

## 10. Appendix

### A. Nmap Commands Reference

```bash
# Full assessment command set
nmap -sn 192.168.1.0/24                          # Host discovery
nmap -sS -sV -O -p- 192.168.1.0/24              # Full TCP scan
nmap -sU --top-ports 100 192.168.1.0/24         # UDP top ports
nmap --script vuln 192.168.1.0/24               # Vulnerability scripts
nmap --script smb-vuln-ms17-010 192.168.1.10    # EternalBlue check
nmap --script ftp-anon 192.168.1.20             # FTP anonymous check
```

### B. Wireshark Display Filters Used

```
telnet                              # Telnet traffic
ftp                                 # FTP commands
ftp-data                            # FTP file transfers
http.request.method == POST         # Form submissions
snmp                                # SNMP queries
tcp.port == 3306                    # MySQL traffic
ip.addr == 192.168.1.20             # Filter by host
```

### C. CVE References

| CVE | Severity | Affected Software |
|-----|----------|------------------|
| CVE-2011-2523 | Critical | vsftpd 2.3.4 backdoor |
| CVE-2016-6515 | High | OpenSSH 7.2p2 DoS |
| CVE-2019-0708 | Critical | RDP BlueKeep |
| MS17-010 | Critical | SMB EternalBlue |

### D. Glossary

| Term | Definition |
|------|-----------|
| Nmap | Network Mapper — open-source port scanner |
| Wireshark | Network protocol analyzer / packet sniffer |
| CVSS | Common Vulnerability Scoring System (0–10 scale) |
| SMB | Server Message Block — Windows file sharing protocol |
| SNMP | Simple Network Management Protocol |
| RDP | Remote Desktop Protocol |
| NLA | Network Level Authentication |
| SFTP | SSH File Transfer Protocol (encrypted) |
| VLAN | Virtual Local Area Network — network segmentation |

---

*Report generated as part of Cybersecurity Internship — Task 10*  
*All assessments performed on authorized lab/test networks only*
