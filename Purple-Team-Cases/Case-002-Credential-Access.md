🔐 Case 002: Credential Access (AS-REP Roasting & Kerberoasting)
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


3. Password Spraying (NetExec)
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
