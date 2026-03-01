# 🏛️ Architecture & Build Log

This section documents the engineering effort required to build the bridge between the on-prem **GOAD lab** and **Microsoft XDR**.

### 🗺️ Network Topology
<img width="1967" height="978" alt="image" src="https://github.com/user-attachments/assets/8733f59f-2f3a-4c42-93b7-1c2a26e678ce" />


### ⚙️ Technical Implementation
1. **Azure Arc**: Integrated DC01, DC02, and DC03 as hybrid resources.
2. **AMA & DCR**: Deployed the **Azure Monitor Agent** and **Data Collection Rules** to stream Event IDs 4624 and 4662 to Sentinel.
3. **XDR Sensors**: Installed **MDI** sensors on Domain Controllers and onboarded workstations to **MDE**.
