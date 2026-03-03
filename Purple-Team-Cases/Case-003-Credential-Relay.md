### Case 003: Credential Relay (LLMNR Poisoning & SMB Relay)
Red Team: Attack Simulation
1. LLMNR/mDNS Poisoning (Responder)
The attacker deployed Responder to listen for broadcast name resolution requests. By spoofing responses for non-existent resources like Bravos.local and Meren.local, the attacker forced victim workstations to attempt authentication against the attacker-controlled machine.

Command: sudo responder -I eth0 -dwv

Result: Successfully sent poisoned answers to WINTERFELL (192.168.58.11), capturing authentication attempts from multiple domain users.

2. NTLMv2 Relaying (ntlmrelayx)
Instead of attempting to crack the captured hashes, the attacker utilized impacket-ntlmrelayx to relay these authentication attempts in real-time to targets previously identified in Case 001 as having SMB Signing Disabled.

Command: ntlmrelayx.py -tf targets.txt -smb2support -socks

Targets: 192.168.58.22 (CASTELBLACK) and 192.168.58.23 (BRAAVOS).

Critical Finding: Successfully relayed the session for NORTH/EDDARD.STARK.

Impact: Gained Full Administrative Access on CASTELBLACK (AdminStatus: TRUE), allowing for lateral movement, credential dumping, or persistent access.

📸 Evidence: Poisoning & Successful Relay
<img width="1914" height="1041" alt="image" src="https://github.com/user-attachments/assets/c8f47ed1-8d18-4f6a-a2df-8af492119889" />
