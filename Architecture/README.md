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



# 4.Defender for Endpoint (MDE): Onboarded the workstation fleet and member servers via local scripts to provide deep EDR visibility into host-level process and network events.
<img width="1914" height="905" alt="image" src="https://github.com/user-attachments/assets/094a356a-5368-4697-ae7f-030364af2012" />


# 5. Microsoft Entra Hybrid Sync
Multi-Domain Identity Bridge: Operationalized Microsoft Entra Connect to synchronize identities across the entire GOAD forest—including sevenkingdoms.local, north.sevenkingdoms.local, and essos.local—with the M365 cloud tenant.

Hybrid Security Core: This integration enables Microsoft Defender XDR to correlate on-premises identity signals (like Kerberoasting) with cloud-based protections (like Conditional Access), forming the analytical backbone of the hybrid SOC.

Real-Time Health Monitoring: Verified consistent synchronization cycles and delta imports to ensure that highly privileged lab identities are accurately represented in the cloud for threat hunting.
<img width="1911" height="954" alt="image" src="https://github.com/user-attachments/assets/ff6d9dc3-bac9-474a-a17b-b079e52d0f49" />
Evidence of active synchronization status and a healthy bridge between the multi-domain GOAD environment and Microsoft Entra ID.

✅ Connectivity Verification
To ensure the integrity of the telemetry pipeline, I executed a Heartbeat query in Microsoft Sentinel. This confirms that the Azure Monitor Agent (AMA) is successfully communicating with the cloud from all five local GOAD virtual machines, bridging the gap between the on-premises VMware environment and the Azure security stack.

```kusto
// Heartbeat check to verify all lab assets are communicating with Azure
Heartbeat
| where TimeGenerated > ago(24h)
| summarize LastContact = max(TimeGenerated) by Computer, Category, OSType
| order by LastContact desc
```
<img width="1573" height="646" alt="image" src="https://github.com/user-attachments/assets/d315f9ad-ba6f-49a0-ae9e-d0afa3f1b025" />


