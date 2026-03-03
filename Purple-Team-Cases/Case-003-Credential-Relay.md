## Case 003: Credential Relay (LLMNR Poisoning & SMB Relay)
Red Team: Attack Simulation
1. LLMNR/mDNS Poisoning (Responder)
The attacker deployed Responder to listen for broadcast name resolution requests. By spoofing responses for non-existent resources like Bravos.local and Meren.local, the attacker forced victim workstations to attempt authentication against the attacker-controlled machine.

Command: sudo responder -I vmnet5

Result: Successfully sent poisoned answers to WINTERFELL (192.168.58.11), capturing authentication attempts from multiple domain users.

### 2. NTLMv2 Relaying (ntlmrelayx)
Instead of attempting to crack the captured hashes, the attacker utilized impacket-ntlmrelayx to relay these authentication attempts in real-time to targets previously identified in Case 001 as having SMB Signing Disabled.

Command: ntlmrelayx.py -tf targets.txt -smb2support -socks

Targets: 192.168.58.22 (CASTELBLACK) and 192.168.58.23 (BRAAVOS).

Critical Finding: Successfully relayed the session for NORTH/EDDARD.STARK.

Impact: Gained Full Administrative Access on CASTELBLACK (AdminStatus: TRUE), allowing for lateral movement, credential dumping, or persistent access.

📸 Evidence: Poisoning & Successful Relay
<img width="1914" height="1041" alt="image" src="https://github.com/user-attachments/assets/c8f47ed1-8d18-4f6a-a2df-8af492119889" />

## 3. Post-Exploitation: Remote Credential Dumping (SOCKS Proxy)
After confirming administrative access via the NTLM relay, a SOCKS4 proxy was leveraged to tunnel advanced post-exploitation tools directly into the target environment without triggering a new authentication event.

Command: proxychains secretsdump.py -no-pass 'NORTH'/'EDDARD.STARK'@192.168.58.22

Technique: OS Credential Dumping (T1003) via an established SOCKS relay.

Objective: Extract the local SAM database, LSA secrets, and cached domain credentials.

Key Findings:
SAM Database Extraction: Successfully dumped local administrator NTLM hashes for persistence.

LSA Secrets & Machine Accounts: Recovered the machine account (CASTELBLACK$) and LSA Secrets, revealing cleartext service credentials.

🚨 CRITICAL DISCOVERY: Extracted the plaintext password for the sql_svc domain account from the LSA secrets:

Username: north.sevenkingdoms.local\sql_svc

Password: YouWillNotKerberoastIngMeeeeee

DPAPI Information: Successfully retrieved DPAPI master keys, enabling the decryption of stored browser credentials or protected files.

📸 Evidence: Secretsdump Output via SOCKS
<img width="1912" height="850" alt="image" src="https://github.com/user-attachments/assets/70693eec-0ace-45fd-b5c0-c8214d6955a1" />

🛡️ Blue Team: Detection & Analysis
1. XDR Incident: LSA Secrets Theft Detection
The execution of secretsdump against CASTELBLACK triggered multiple high-fidelity alerts in the Microsoft 365 Defender portal.

Alert Name: Indication of local security authority secrets theft.

Alert Name: Compromised account conducting hands-on-keyboard attack.

Detection Source: Endpoint Detection and Response (EDR).

Technical Evidence: The portal identified that ntoskrnl.exe opened the Windows Remote Registry Service named pipe (winreg), a behavior directly associated with remote credential dumping tools.

📸 Evidence: Defender XDR Behavioral Alert
<img width="1915" height="953" alt="image" src="https://github.com/user-attachments/assets/8f3caa6e-502f-40dc-bf99-505946d79400" />
Defender XDR alert story showing the process tree and the detection of LSA secrets theft on CASTELBLACK.
