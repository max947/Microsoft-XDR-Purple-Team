
# 🕵️ Case 001: Network Reconnaissance & SMB Enumeration

## ⚔️ Red Team: Attack Simulation

### **1. Aggressive Service Enumeration (Nmap)**
The attacker initiated an aggressive service and script scan to map the internal attack surface and identify high-value targets.
* **Command**: `nmap -sC -sV 192.168.58.0/24`
* **Objective**: Identify open ports, service versions, and run default scripts to discover OS versions and SMB vulnerabilities.
* **Impact**: This scan identified multiple Domain Controllers (`KINGSLANDING`, `WINTERFELL`, `ESSOS`) and verified open services like SMB (445), LDAP (389), and RDP (3389).

> **<img width="1919" height="1020" alt="image" src="https://github.com/user-attachments/assets/a540b80a-04f5-4159-a94b-3531f207a2b6" />
**

### 2. Domain Metadata Harvesting (NetExec)
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
Following the initial scan, NetExec (nxc) was utilized to enumerate the sevenkingdoms.local forest structure and harvest active domain user lists.

Command: nxc smb 192.168.58.0/24 --users

Objective: Enumerate active users and extract metadata from Active Directory object descriptions.

Key Findings:

Plaintext Credential Leak: Discovered a critical security misconfiguration where a user password was stored in the AD Description field for samwell.tarly (Password: Heartsbane).

User Harvesting: Successfully enumerated 10 local users from the NORTH domain, providing a roadmap for the next phase of the attack.

Vulnerability Identification: Confirmed BRAAVOS and CASTELBLACK have SMB Signing Disabled, which, combined with the leaked credentials, enables immediate lateral movement.

📸 Evidence: Credential Leak & User Enumeration
<img width="1920" height="411" alt="image" src="https://github.com/user-attachments/assets/dca5cbbd-2de5-46ea-996c-79e3042f1edd" />

Gemini said
Based on your latest screenshot (image_64e016.png), you have transitioned from simple user harvesting to Share Enumeration using a Null Session/Guest access strategy. This is a critical step in a "Purple Team" exercise as it identifies where an attacker can read or write data without needing valid domain credentials.

Below is the copy-paste-ready Markdown for Section 4 of your Case-001-Network-Recon.md file.

### 4. Unauthenticated Share Enumeration (NetExec)
To identify potential data exfiltration points or lateral movement vectors, NetExec (nxc) was used to scan the network for accessible SMB shares using a null/guest user session.

Command: nxc smb 192.168.58.10-23 -u 'a' -p '' --shares

Objective: Identify shares with READ or READ/WRITE permissions accessible to unauthenticated or guest users.

Key Findings:

Guest Access Identified: Successfully gained (Guest) access to BRAAVOS (192.168.58.23) and CASTELBLACK (192.168.58.22).

Sensitive Share Exposure:

Identified a public share on BRAAVOS with READ access for all domain users.

Discovered an all share with READ,WRITE permissions on both vulnerable hosts, providing a high-risk vector for malware deployment or data tampering.

Administrative Exposure: Verified that standard administrative shares (C$, ADMIN$, IPC$) are visible, though restricted, confirming the targets are Windows-based systems.

📸 Evidence: Unauthenticated Share Discovery
<img width="1912" height="643" alt="image" src="https://github.com/user-attachments/assets/47081218-671a-4501-933b-f5bd0e652c79" />

### 🛡️ Blue Team: Remediation & Hardening
1. Disabling Legacy Protocols (LLMNR/NetBIOS)
The attacker relied on the network's ability to broadcast identity requests. Disabling these legacy protocols prevents an attacker from "poisoning" name resolution to intercept credentials.

Action: Disable LLMNR (Link-Local Multicast Name Resolution) via Group Policy: Computer Configuration -> Administrative Templates -> Network -> DNS Client -> Turn off multicast name resolution.

Action: Disable NetBIOS over TCP/IP on all network adapters via DHCP options or manual configuration.

2. Enforcing SMB Signing (Mitigating Relay)
The discovery of signing:False on BRAAVOS and CASTELBLACK is a critical vulnerability that allows NTLM Relay attacks.

Action: Enable SMB Signing Required via GPO for both Clients and Servers: Microsoft network server: Digitally sign communications (always) set to Enabled.

Impact: This cryptographically signs SMB packets, ensuring that an attacker cannot intercept and "relay" a session to another machine.

3. Restricting Unauthenticated (Null) Sessions
The attacker successfully used a "null session" strategy (-u 'a' -p '') to enumerate shares on CASTELBLACK.

Action: Restrict anonymous access to SMB shares and named pipes via Security Options: Network access: Restrict anonymous access to Named Pipes and Shares set to Enabled.

Action: Ensure Network access: Do not allow anonymous enumeration of SAM accounts and shares is set to Enabled.

4. Cleaning Active Directory Metadata
The leak of the password Heartsbane in the samwell.tarly description field is a major human-error risk.

Action: Run a PowerShell audit script to scan all Active Directory object "Description" and "Notes" fields for keywords like "Password", "PWD", or "Credentials".

Action: Implement a Service Account Policy that forbids the storage of any sensitive data in non-secret attributes.

5. Network Segmentation & Monitoring
Segmentation: Place high-value assets like WINTERFELL (DC) and database servers in isolated VLANs to prevent lateral scanning from standard workstations.

SOC Alerting: Configure Microsoft Sentinel to alert on internal "Port Sweeps" (Event ID 4624/4625 spikes) originating from a single internal IP.
