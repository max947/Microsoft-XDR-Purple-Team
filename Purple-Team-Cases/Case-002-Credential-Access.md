### Case 002: Credential Access (AS-REP Roasting & Kerberoasting)
⚔️ Red Team: Attack Simulation
1. AS-REP Roasting (Impacket)
Using the username list harvested in Case 001, an AS-REP Roast was performed to identify accounts vulnerable to credential harvesting without authentication.

Command: GetNPUsers.py north.sevenkingdoms.local/ -no-pass -usersfile users.txt

Target: brandon.stark@NORTH.SEVENKINGDOMS.LOCAL

Finding: The account was identified as having UF_DONT_REQUIRE_PREAUTH set, allowing for the extraction of a crackable Kerberos hash.

2. Offline Password Cracking (Hashcat)
The harvested hash was subjected to an offline dictionary attack to recover the plaintext password.

Command: hashcat -m 18200 asrep.hash /opt/wordlist/rockyou.txt

Result: CRACKED

Recovered Password: iseedeadpeople

📸 Evidence: Successful Credential Recovery
<img width="1919" height="596" alt="image" src="https://github.com/user-attachments/assets/ee42923a-fed5-4235-a588-75ae456fd5d3" />

🛡️ Blue Team: Detection & Analysis
1. XDR Incident Validation
The AS-REP Roasting attack against brandon.stark was immediately identified by Microsoft Defender for Identity.

Alert Name: Suspicious Kerberos authentication (AS-REQ)

Detection Source: Microsoft Defender for Identity

MITRE ATT&CK Mapping: T1558.004 (AS-REP Roasting)

Evidence: The portal flagged multiple anomalous TGT requests originating from the attacker machine (192.168.58.1), providing clear visibility into the credential harvesting attempt.

📸 Evidence: M365 Defender Security Alert
<img width="1890" height="916" alt="image" src="https://github.com/user-attachments/assets/40248626-0505-4bc2-a02d-6f390c0fc7ae" />
### SOC Analyst: Incident Handling & Remediation
1. Threat Classification & Validation
Upon receiving the AS-REP Roasting alert, the incident was investigated and validated as a True Positive.

Classification: Marked as True Positive -> Multi-staged attack (image_bda1fd.png).

Impact Assessment: Confirmed that the account north.sevenkingdoms.local\brandon.stark was vulnerable due to missing Kerberos pre-authentication, leading to a successful offline password crack.

2. Identity Containment & Recovery
To prevent further exploitation and lateral movement, the following identity containment actions were executed immediately:

Account Disabling: Disabled the brandon.stark account in Active Directory to kill all current on-premises sessions.

Entra ID Synchronization: Verified the account status in Microsoft Entra ID and disabled the cloud identity to block access to integrated SaaS applications.

Credential Invalidation: Triggered a Force Password Change on the next logon, ensuring the attacker's current plaintext password is no longer valid.

3. Evidence of Success
XDR Alert State: The alert status has been updated to In Progress and assigned to a specialized analyst for final review.

Detection Efficacy: The Microsoft Defender for Identity sensor correctly identified the anomalous AS-REQ requests, providing the precise timestamp and source IP (192.168.58.1) required for host isolation.

4. Long-Term Hardening (Post-Incident)
Vulnerability Eradication: Conducted a domain-wide audit to ensure no other accounts remain configured with UF_DONT_REQUIRE_PREAUTH.

Conditional Access: Proposed a new Conditional Access Policy requiring MFA for all logins originating from non-compliant devices to prevent future credential-only compromises.
<img width="1919" height="954" alt="image" src="https://github.com/user-attachments/assets/ac0ad024-3d5c-43f3-bf6f-b6cfad085835" />

### 3. Password Spraying (NetExec)
Using the username list harvested during the reconnaissance phase, a targeted password spray was conducted to identify accounts utilizing weak or default credentials.

Command: nxc smb 192.168.58.11 -u users.txt -p users.txt --no-bruteforce

Objective: Identify accounts where the password matches the username, a common occurrence in lab environments and misconfigured production domains.

Result: SUCCESS. Identified that the account north.sevenkingdoms.local\hodor is using the password hodor.

📸 Evidence: Successful Password Spray
<img width="1918" height="276" alt="image" src="https://github.com/user-attachments/assets/0007bea8-11bc-4c05-a71f-82f90b289e71" />

4. Kerberoasting (NetExec & Hashcat)
Following the initial identity harvesting, an authenticated Kerberoasting attack was executed using the compromised brandon.stark credentials to target service accounts with registered Service Principal Names (SPNs).

Command: nxc ldap 192.168.58.11 -u brandon.stark -p 'iseedeadpeople' --kerberoasting KERBEROASTING

Objective: Retrieve encrypted Ticket Granting Service (TGS) tickets for offline cracking.

Findings: Successfully harvested TGS hashes for high-value targets including jon.snow, sansa.stark, and sql_svc.

Cracking Result: Utilized Hashcat to perform a dictionary attack against the jon.snow hash.

Command:  hashcat -m 13100 --force -a 0 KERBEROASTING /opt/wordlist/rockyou.txt

Result: CRACKED — Password recovered: iknownothing

📸 Evidence: Authenticated Kerberoasting Results

<img width="1919" height="937" alt="image" src="https://github.com/user-attachments/assets/a853b2a2-56f0-48a4-beae-31c6ff62450b" />

<img width="1920" height="244" alt="image" src="https://github.com/user-attachments/assets/016f8703-8d61-49e0-909e-a059c785f6fa" />

🛡️ Blue Team: Detection & Analysis
1. Defender XDR Incident: Identity Escalation
The Microsoft Defender for Identity sensor flagged the entire Kerberoasting lifecycle in real-time.

Alert 1: Possible Kerberoasting LDAP reconnaissance: Triggered when the attacker queried for service principal names (SPNs).

Alert 2: Suspected Kerberos SPN exposure: Triggered during the ticket harvesting phase.

Alert 3: Possible Kerberoasting attack: The final correlated alert confirming the attack intent.

📸 Evidence: Defender for Identity High-Severity Alerts
<img width="1916" height="358" alt="image" src="https://github.com/user-attachments/assets/65a298b3-0ec8-4cd5-95ef-2ac6a6f1d09a" />

### Incident Handling & Neutralization
As a SOC Analyst, immediate action was taken to contain the threat. Because Kerberoasting is an offline attack, the hashes were likely already exfiltrated; therefore, containment focused on account invalidation.

Disable Source Account: The brandon.stark account used for reconnaissance was immediately Disabled in Active Directory and Entra ID.

Forced Password Resets: Triggered immediate password resets for targeted service accounts (sql_svc, sansa.stark, and jon.snow) to invalidate any passwords cracked offline.

Host Isolation: Utilized Microsoft Defender for Endpoint (MDE) to "Isolate Device" for the source host (192.168.58.1), severing all network communication.

📸 Evidence: Final Resolution & Account Containment
<img width="1500" height="400" alt="image" src="https://github.com/user-attachments/assets/image_bd2e39.png" />
Figure 2: User profile view confirming the account status is "Disabled" and marked as "Compromised".

🏁 Hardening & Long-Term Remediation
To prevent a recurrence, the following domain-wide architectural hardening steps were implemented:

Strengthen Service Account Passwords:

Enforce Length: Updated Domain Group Policy to require service account passwords to be at least 25+ characters to resist dictionary attacks like rockyou.txt.

Deploy Managed Service Accounts (gMSA):

The Ultimate Fix: Transitioned the sql_svc account to a Group Managed Service Account (gMSA).

Impact: gMSA passwords are 240 characters long and managed by the Domain Controller, making them immune to offline cracking.

Audit SPN Exposure:

Least Privilege: Restricted LDAP searching permissions to prevent standard users from enumerating the servicePrincipalName attribute.
