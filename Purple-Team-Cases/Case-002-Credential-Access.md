## Case 002: Credential Access (AS-REP Roasting & Kerberoasting)
⚔️ Red Team: Attack Simulation
1. AS-REP Roasting (Impacket)
Using the username list harvested in Case 001, an AS-REP Roasting attack was performed to identify accounts vulnerable to credential harvesting without requiring prior authentication.

Command: GetNPUsers.py north.sevenkingdoms.local/ -no-pass -usersfile users.txt

Target: brandon.stark@NORTH.SEVENKINGDOMS.LOCAL

Finding: The account was identified with the UF_DONT_REQUIRE_PREAUTH flag set, allowing the extraction of a crackable Kerberos hash.

2. Offline Password Cracking (Hashcat)
The harvested hash was subjected to an offline dictionary attack to recover the plaintext password.

Command: hashcat -m 18200 asrep.hash /opt/wordlist/rockyou.txt

Result: CRACKED

Recovered Password: iseedeadpeople

📸 Evidence: Successful Credential Recovery
<img width="1919" height="596" alt="image" src="https://github.com/user-attachments/assets/ee42923a-fed5-4235-a588-75ae456fd5d3" />

🛡️ Blue Team: Detection & Analysis
1. XDR Incident Validation
The attack against brandon.stark was immediately flagged by Microsoft Defender for Identity (MDI).

Alert Name: Suspicious Kerberos authentication (AS-REQ)

Detection Source: Microsoft Defender for Identity

MITRE ATT&CK Mapping: T1558.004 (AS-REP Roasting)

Analysis: The portal flagged multiple anomalous TGT requests originating from the attacker machine (192.168.58.1), providing clear visibility into the harvesting attempt.

📸 Evidence: M365 Defender Security Alert
<img width="1890" height="916" alt="image" src="https://github.com/user-attachments/assets/40248626-0505-4bc2-a02d-6f390c0fc7ae" />

🕵️ SOC Analyst: Incident Handling & Remediation
1. Threat Classification
Upon investigation, the incident was validated as a True Positive.

Classification: Multi-staged Identity Attack.

Impact Assessment: Confirmed brandon.stark was vulnerable due to missing Kerberos pre-authentication, leading to a successful credential compromise.

2. Identity Containment & Recovery
To halt lateral movement, the following actions were executed:

Account Disabling: Disabled brandon.stark in Active Directory to terminate all active on-premises sessions.

Entra ID Sync: Verified status in Microsoft Entra ID and disabled the cloud identity to block SaaS application access.

Credential Invalidation: Triggered a Force Password Change on next logon, rendering the cracked password useless.

3. Long-Term Hardening
Vulnerability Eradication: Conducted a domain-wide audit to ensure no other accounts remain configured with UF_DONT_REQUIRE_PREAUTH.

Conditional Access: Proposed a policy requiring MFA for all logins from non-compliant devices.

📸 Evidence: Remediation Status
<img width="1919" height="954" alt="image" src="https://github.com/user-attachments/assets/ac0ad024-3d5c-43f3-bf6f-b6cfad085835" />

⚔️ Red Team: Phase 2 (Lateral Movement)
3. Password Spraying (NetExec)
Using the initial reconnaissance list, a targeted password spray was conducted to find weak or default credentials.

Command: nxc smb 192.168.58.11 -u users.txt -p users.txt --no-bruteforce

Result: SUCCESS. Account north.sevenkingdoms.local\hodor was identified using the password hodor.

📸 Evidence: Successful Password Spray
<img width="1918" height="276" alt="image" src="https://github.com/user-attachments/assets/0007bea8-11bc-4c05-a71f-82f90b289e71" />

4. Kerberoasting (NetExec & Hashcat)
Using the compromised brandon.stark credentials, an authenticated Kerberoasting attack targeted high-value service accounts.

Command: nxc ldap 192.168.58.11 -u brandon.stark -p 'iseedeadpeople' --kerberoasting KERBEROASTING

Targets: jon.snow, sansa.stark, and sql_svc.

Cracking Result: CRACKED — Password for jon.snow recovered: iknownothing.

📸 Evidence: Authenticated Kerberoasting Results
<img width="1919" height="937" alt="image" src="https://github.com/user-attachments/assets/a853b2a2-56f0-48a4-beae-31c6ff62450b" />

🛡️ Blue Team: Response & Neutralization
1. Defender XDR Correlation
The MDI sensor flagged the Kerberoasting lifecycle in real-time across three distinct stages:

LDAP Reconnaissance: Querying for Service Principal Names (SPNs).

SPN Exposure: Unusual ticket requests for service accounts.

Kerberoasting Attack: Final correlated alert confirming malicious intent.

📸 Evidence: Defender for Identity High-Severity Alerts
<img width="1916" height="358" alt="image" src="https://github.com/user-attachments/assets/65a298b3-0ec8-4cd5-95ef-2ac6a6f1d09a" />

2. Rapid Containment Actions
Disable Source Account: brandon.stark was immediately disabled in AD and Entra ID.

Forced Password Resets: Immediate resets for sql_svc, sansa.stark, and jon.snow to invalidate exfiltrated hashes.

Host Isolation: Utilized Microsoft Defender for Endpoint (MDE) to isolate the attacker's host (192.168.58.1).

📸 Evidence: User Status - Disabled & Compromised
<img width="1500" height="400" alt="image" src="https://github.com/user-attachments/assets/image_bd2e39.png" />

🏁 Final Hardening & Remediation
To prevent recurrence, the domain architecture was updated:

Managed Service Accounts (gMSA): Transitioned sql_svc to a gMSA. These use 240-character, automatically managed passwords that are immune to offline cracking.

Password Complexity: Updated GPO to require 25+ characters for any remaining service accounts.

Least Privilege: Restricted LDAP search permissions to prevent standard users from enumerating the servicePrincipalName (SPN) attribute.
