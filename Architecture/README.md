# 🏛️ Architecture & Build Log

This section documents the engineering effort required to build the bridge between the on-prem **GOAD lab** and the **Microsoft Defender XDR** stack.

### 🗺️ Network Topology
<img width="100%" alt="GOAD Lab Network Topology" src="https://github.com/user-attachments/assets/8733f59f-2f3a-4c42-93b7-1c2a26e678ce" />

> **Note:** This environment is isolated on the `192.168.58.0/24` subnet. It simulates a complex, multi-forest corporate network across three distinct domains: `sevenkingdoms.local`, `north.sevenkingdoms.local`, and `essos.local`.

---

## ⚙️ Technical Implementation

### 🔹 1. Azure Arc Integration
* **Hybrid Management:** Successfully onboarded all lab domain controllers (**DC01, DC02, DC03**) as Azure Arc-enabled servers.
* **Impact:** This enables centralized governance and allows Azure-native security policies and monitoring to be applied to local VMware-hosted infrastructure.

<img width="100%" alt="Azure Arc Server Inventory" src="https://github.com/user-attachments/assets/ec2c60cd-87b1-4e5e-a3d5-17ded000db17" />

### 🔹 2. AMA & DCR Configuration
* **Azure Monitor Agent (AMA):** Deployed the AMA across the fleet to facilitate high-speed, secure telemetry streaming.
* **Data Collection Rules (DCR):** Engineered granular DCRs to filter and stream high-value Windows Event Logs (e.g., **4624** for Logons, **4662** for AD Object Access) into the Microsoft Sentinel workspace.

<img width="100%" alt="DCR Visualizer Flow" src="https://github.com/user-attachments/assets/8e7a8456-2bd6-46be-9584-b238a62bd1ea" />

### 🔹 3. Microsoft XDR Sensor Deployment
* **Defender for Identity (MDI):** Operationalized MDI sensors on all Domain Controllers. Configured **Group Managed Service Accounts (GMSA)** to allow sensors to parse and monitor AD identity traffic.
* **Defender for Endpoint (MDE):** Onboarded the workstation fleet and member servers via local scripts to provide deep EDR visibility into host-level process and network events.

<img width="100%" alt="MDI Sensor Health" src="https://github.com/user-attachments/assets/5f94db28-3aeb-4c43-ac43-b95e65c8e161" />
<img width="100%" alt="MDE Device Inventory" src="https://github.com/user-attachments/assets/094a356a-5368-4697-ae7f-030364af2012" />

### 🔹 4. Microsoft Entra Hybrid Sync
* **Multi-Domain Bridge:** Operationalized **Microsoft Entra Connect** to synchronize identities across the entire GOAD forest (3 domains) with the M365 cloud tenant.
* **Security Backbone:** This enables **Microsoft Defender XDR** to correlate on-premises signals (like Kerberoasting) with cloud-based protections (like Conditional Access).
* **Verification:** Monitored consistent synchronization cycles to ensure high-privileged lab identities are accurately represented in the cloud.

<img width="100%" alt="Entra Sync Status" src="https://github.com/user-attachments/assets/ff6d9dc3-bac9-474a-a17b-b079e52d0f49" />
*Evidence of a healthy synchronization bridge between the multi-domain GOAD environment and Microsoft Entra ID.*

---

## ✅ Connectivity Verification
To ensure the integrity of the telemetry pipeline, I executed a **Heartbeat** query in **Microsoft Sentinel**. This confirms that the **Azure Monitor Agent (AMA)** is successfully communicating with the cloud from all local GOAD virtual machines.

```kusto
// Heartbeat check to verify all lab assets are communicating with Azure
Heartbeat
| where TimeGenerated > ago(24h)
| summarize LastContact = max(TimeGenerated) by Computer, Category, OSType
| order by LastContact desc
```

<img width="1908" height="857" alt="image" src="https://github.com/user-attachments/assets/f5ec16cd-f35b-4ea6-822c-38214abe6312" />

