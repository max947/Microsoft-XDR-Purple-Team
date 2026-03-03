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
<img width="1618" height="910" alt="image" src="https://github.com/user-attachments/assets/7fb6a421-10b1-47cf-bbac-01683178fef7" />

3. Password Spraying (NetExec)
Using the username list harvested during the reconnaissance phase, a targeted password spray was conducted to identify accounts utilizing weak or default credentials.

Command: nxc smb 192.168.58.11 -u users.txt -p users.txt --no-bruteforce

Objective: Identify accounts where the password matches the username, a common occurrence in lab environments and misconfigured production domains.

Result: SUCCESS. Identified that the account north.sevenkingdoms.local\hodor is using the password hodor.

📸 Evidence: Successful Password Spray
