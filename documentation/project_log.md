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

## Phase 3 — Hardening (Pending) (TO BE COMPLETED TOMORROW)
- Apply CIS Benchmark controls
- Patch MS17-010
- Enable SMB signing
- Disable unnecessary services
- Re-run exploit to confirm it is blocked




















---
