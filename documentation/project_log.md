# Secure Baseline Lab — Project Log

## Project Overview
- **Target:** Windows Server 2008 R2 (192.168.56.3)
- **Attacker:** Kali Linux (192.168.56.4)
- **Standard:** CIS Benchmark
- **Start Date:** 11/05/2026

---

## Phase 1 — Initial Scanning

### Nmap Scan
- **Date:** 11/05/2026
- **Command:** nmap -sV -sC -O 192.168.56.3 -oN scans/initial_nmap_scan.txt
- **Purpose:** Identify all open ports and services on the target before any hardening

### Findings
- Port 135 — Microsoft RPC open (information leakage risk)
- Port 139 — NetBIOS open (legacy protocol, info leakage)
- Port 445 — SMB open, message signing DISABLED (critical)
- Port 49154 — Dynamic RPC port open
- Guest account able to query SMB with no credentials
- Computer name exposed: WINDOW

## Phase 2 — Exploitation

### MS17-010 EternalBlue
- **Date:** 11/05/2026
- **Tool:** Metasploit Framework
- **Module:** exploit/windows/smb/ms17_010_eternalblue
- **Target:** 192.168.56.3:445
- **Result:** SUCCESS
- **Access gained:** NT AUTHORITY\SYSTEM
- **Evidence:** screenshots/eternalblue_exploit.png
- **Evidence:** screenshots/system_access_proof.png

### What this means
Port 445 (SMB) was open and unpatched. EternalBlue exploited a 
buffer overflow vulnerability in the SMBv1 protocol to gain 
unauthenticated SYSTEM level access in seconds. This is the same 
exploit used in the 2017 WannaCry ransomware attack.

## Phase 3 — Hardening 

### Environment
- Target: Windows Server 2008 R2 (192.168.56.3)
- Attacker: Kali Linux (192.168.56.4)
- Date: 13/05/2026

### Pre-Hardening Snapshot
- VirtualBox snapshot taken before any changes: 'Pre-Hardening Baseline'

---

### Control 1 — Disable SMBv1
- **Method:** Registry key `SMB1 = 0` under LanmanServer\Parameters
- **Command:** `Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" -Name SMB1 -Type DWORD -Value 0 -Force`
- **Verification:** `Get-ItemProperty` confirmed value = 0
- **Evidence:** screenshots/SMB1-Before.png, screenshots/SMB1-After.png

---

### Control 2 — Enable SMB Signing
- **Method:** Registry — EnableSecuritySignature + RequireSecuritySignature = 1
- **Applied to:** LanmanServer and LanmanWorkstation
- **Verification:** Both values confirmed as 1 via PowerShell
- **Evidence:** screenshots/SMB-Signing-After.png

---

### Control 3 — Apply MS17-010 Patch (KB4012212)
- **Patch:** KB4012212
- **Pre-requisite:** Service Pack 1 (KB976932) required first — installed before patch
- **Verification:** `Get-HotFix -Id KB4012212` confirmed installed 13/05/2026
- **Evidence:** screenshots/kb4012212-installed.png

---

### Control 4 — Enable Windows Firewall
- **Method:** `netsh advfirewall set allprofiles state on`
- **Additional rule:** Inbound TCP port 445 blocked
- **Verification:** All three profiles (Domain, Private, Public) confirmed ON
- **Evidence:** screenshots/firewall-enabled.png

---

### Post-Hardening Verification (from Kali)

| Test | Pre-Hardening | Post-Hardening |
|------|--------------|----------------|
| MS17-010 (Metasploit) | VULNERABLE | CONNECTION TIMEOUT |
| Port 445 (Nmap) | Open | Filtered |
| SMBv1 (Nmap) | Enabled | Disabled |
| SMB Signing | Not required | Required |
| Windows Firewall | Off | On |

- **Evidence:** screenshots/metasploit-not-vulnerable.png
- **Evidence:** screenshots/nmap-port-445-filtered.png
- **Evidence:** screenshots/nmap-smb-protocols.png

---

### Troubleshooting & Lessons Learned
- Windows Server 2008 R2 on a Host-Only network has no internet access — patch downloaded on host machine and transferred via VirtualBox Shared Folder
- VirtualBox Guest Additions required before shared folders work
- KB4012212 requires Service Pack 1 (KB976932) to be installed first
- VM disk required resizing from 15GB to 30GB before Service Pack 1 could be installed
- Disk resize required running VBoxManage on the snapshot file, not just the base VDI

## Phase 4 — Web Application Vulnerability Testing

### Environment
- Attacker: Kali Linux (192.168.56.4)
- Target: DVWA (Damn Vulnerable Web Application)
- Date: 13/05/2026

### Setup
- Installed Apache2, PHP and MySQL on Kali Linux
- Cloned DVWA from GitHub to /var/www/html/dvwa
- Configured DVWA database and user credentials
- Confirmed DVWA accessible at http://127.0.0.1/dvwa
- Security level set to Low for initial testing

### Evidence
- screenshots/dvwa-setup-page.png
- screenshots/dvwa-home.png
- screenshots/dvwa-security-low.png

### Vulnerabilities to Test
- SQL Injection
- XSS (Reflected, Stored, DOM)
- CSRF
- Command Injection
- File Inclusion
- Brute Force


















---
