
# 🕵️ Case 001: Network Reconnaissance & SMB Enumeration

## ⚔️ Red Team: Attack Simulation

### **1. Aggressive Service Enumeration (Nmap)**
The attacker initiated an aggressive service and script scan to map the internal attack surface and identify high-value targets.
* **Command**: `nmap -sC -sV 192.168.58.0/24`
* **Objective**: Identify open ports, service versions, and run default scripts to discover OS versions and SMB vulnerabilities.
* **Impact**: This scan identified multiple Domain Controllers (`KINGSLANDING`, `WINTERFELL`, `ESSOS`) and verified open services like SMB (445), LDAP (389), and RDP (3389).

> **<img width="1919" height="1020" alt="image" src="https://github.com/user-attachments/assets/a540b80a-04f5-4159-a94b-3531f207a2b6" />
**

### **2. Domain Metadata Harvesting (NetExec)
Following the initial port scan, NetExec (nxc) was utilized to perform unauthenticated enumeration across the .58 subnet to map the sevenkingdoms.local forest structure and identify misconfigurations.

Command: nxc smb 192.168.58.0/24

Objective: Enumerate domain names, OS versions, and SMB Signing status to identify potential relay targets.

Key Findings:

Forest Mapping: Successfully mapped three distinct domains: sevenkingdoms.local, north.sevenkingdoms.local, and essos.local.

Vulnerability Identification: Identified BRAAVOS (192.168.58.23) and CASTELBLACK (192.168.58.22) with SMB Signing Disabled (signing:False), making them prime targets for NTLM Relay attacks.

Legacy Protocols: Detected SMBv1 enabled on multiple targets (e.g., MEEREEN, BRAAVOS), significantly increasing the attack surface.

📸 Evidence: NetExec Enumeration Results
<img width="1920" height="215" alt="image" src="https://github.com/user-attachments/assets/5b1ac58a-16bd-4b66-94ee-fba1d676b550" />

### 3. Domain Metadata Harvesting (NetExec)
Following the initial port scan, NetExec (nxc) was utilized to perform unauthenticated enumeration across the .58 subnet to map the sevenkingdoms.local forest structure and identify exploitable misconfigurations.

Command: nxc smb 192.168.58.0/24 --users

Objective: Enumerate domain names, OS versions, SMB signing status, and harvest active domain user lists.

Key Findings:

User Enumeration: Successfully enumerated 10 local users from the NORTH domain, including high-value targets like jon.snow and arya.stark.

Vulnerability Identification: Confirmed BRAAVOS (192.168.58.23) and CASTELBLACK (192.168.58.22) have SMB Signing Disabled (signing:False).

Protocol Weakness: Detected SMBv1 and Null Authentication enabled on multiple targets, providing an immediate path for lateral movement.

Evidence:

<img width="1920" height="215" alt="image" src="https://github.com/user-attachments/assets/5b1ac58a-16bd-4b66-94ee-fba1d676b550" />

