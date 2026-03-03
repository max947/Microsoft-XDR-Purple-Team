🔐 Case 002: Credential Access (AS-REP Roasting & Kerberoasting)
⚔️ Red Team: Attack Simulation
1. AS-REP Roasting (Impacket)
Using the username list harvested in Case 001, the attacker performed an AS-REP Roast to identify accounts where Kerberos pre-authentication is not required.

Command: GetNPUsers.py north.sevenkingdoms.local/ -no-pass -usersfile users.txt

Objective: Retrieve an encrypted Kerberos AS-REP ticket for offline password cracking.

Key Finding: Successfully retrieved a hash for brandon.stark@NORTH.SEVENKINGDOMS.LOCAL. All other users in the list correctly have pre-authentication enforced.

📸 Evidence: Successful AS-REP Ticket Harvesting
<img width="1915" height="474" alt="image" src="https://github.com/user-attachments/assets/758a9d47-f8b8-4a0f-905e-5a818bef98fe" />
