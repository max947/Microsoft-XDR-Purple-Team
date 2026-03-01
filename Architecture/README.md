# 🏛️ Architecture & Build Log

This section documents the engineering effort required to build the bridge between the on-prem **GOAD lab** and **Microsoft XDR**.

### 🗺️ Network Topology
<img width="1967" height="978" alt="image" src="https://github.com/user-attachments/assets/8733f59f-2f3a-4c42-93b7-1c2a26e678ce" />
Note: This environment is configured on the 192.168.58.0/24 subnet for environment isolation.

⚙️ Technical Implementation
🔹 1. Azure Arc Integration
Hybrid Management: Successfully onboarded DC01, DC02, and DC03 as Azure Arc-enabled servers.

Centralized Governance: Integration with Arc allows for the application of Azure-native security policies and monitoring to the local VMware-hosted infrastructure.

<img width="1898" height="519" alt="image" src="https://github.com/user-attachments/assets/ec2c60cd-87b1-4e5e-a3d5-17ded000db17" />


🔹 2. AMA & DCR Configuration
Azure Monitor Agent (AMA): Deployed the AMA across all lab assets to facilitate high-speed, secure telemetry streaming.

Data Collection Rules (DCR): Engineered granular DCRs to filter and stream high-value Windows Event Logs (e.g., 4624 for Logons, 4662 for AD Object Access) into the Microsoft Sentinel workspace.

<img width="1917" height="721" alt="image" src="https://github.com/user-attachments/assets/8e7a8456-2bd6-46be-9584-b238a62bd1ea" />


🔹 3. Microsoft XDR Sensor Deployment
Defender for Identity (MDI): Operationalized MDI sensors on all Domain Controllers. Configured Directory Service Accounts (GMSA) to allow the sensors to parse and monitor AD identity traffic.
<img width="1908" height="952" alt="image" src="https://github.com/user-attachments/assets/5f94db28-3aeb-4c43-ac43-b95e65c8e161" />



Defender for Endpoint (MDE): Onboarded the workstation fleet and member servers via local scripts to provide deep EDR visibility into host-level process and network events.
<img width="1914" height="905" alt="image" src="https://github.com/user-attachments/assets/094a356a-5368-4697-ae7f-030364af2012" />
