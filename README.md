# Microsoft-XDR-Purple-Team
The goal of this project is to bridge the gap between offensive techniques and defensive engineering. By deploying the Game of Active Directory (GOAD) in a VMware environment and onboarding it to the Microsoft XDR stack, I have created a continuous loop of attack simulation and detection validation.

## 🚀 Project Overview
The goal of this project is to bridge the gap between offensive techniques and defensive engineering. By deploying the **Game of Active Directory (GOAD)** in a VMware environment and onboarding it to the **Microsoft XDR** stack, I have created a continuous loop of attack simulation and detection validation.

---

## 🏗️ Architecture & Implementation
This lab simulates a complex, multi-forest corporate network across three distinct domains.

### **1. Infrastructure (On-Premise)**
* **Lab Framework:** GOAD v3 deployed via **VMware Workstation**.
* **Network Subnet:** `192.168.58.0/24` (Configured manually to avoid subnet conflicts).
* **Active Directory:** 5+ Windows Servers (2016/2019/2022) featuring forest trusts and intentionally vulnerable configurations.

### **2. Security Stack (Cloud-Integrated)**
* **Microsoft Defender for Identity (MDI):** Sensors operationalized on all Domain Controllers (**DC01, DC02, DC03**) to monitor LDAP, Kerberos, and NTLM telemetry.
* **Microsoft Defender for Endpoint (MDE):** Onboarded workstations and member servers via local scripts to ingest deep EDR telemetry.
* **Microsoft Sentinel (SIEM):** Centralized Log Analytics Workspace (LAW) configured to ingest XDR alerts and raw data for custom hunting.

[Image of Microsoft Sentinel and Defender for Identity data flow diagram]

---

## ⚔️ Purple Team Workflow
I utilize a structured lifecycle for every threat emulated in this lab:

1.  **Emulation:** Execute attack techniques (e.g., **Kerberoasting**, **DCSync**) using tools like **Rubeus**, **Mimikatz**, and **Atomic Red Team**.
2.  **Telemetry Analysis:** Observe the digital footprint in the **Defender XDR** portal and **Sentinel** logs (`DeviceProcessEvents`, `IdentityDirectoryEvents`).
3.  **Detection Engineering:** Identify "Detection Gaps" where default alerts failed and develop custom **KQL** (Kusto Query Language) rules to bridge the gap.
4.  **Response Automation:** Architect **Azure Logic Apps** (Playbooks) to automate the isolation of compromised hosts or the disabling of accounts upon alert trigger.

---

## 📊 Detection Journal (Key Findings)
| Attack Technique | MITRE ID | Sensor | Alert Triggered? | Custom KQL Written? |
| :--- | :--- | :--- | :--- | :--- |
| **Nmap Reconnaissance** | `T1595.001` | MDE | No (Threshold Gap) | **Yes (View Query)** |
| **Kerberoasting** | `T1558.003` | MDI | Yes | No (Default) |
| **DCSync** | `T1003.006` | MDI | Yes | **Yes (Automation)** |

---

## 🛠️ Technical Skills Demonstrated
* **Security Operations:** SIEM/SOAR Management, Incident Triage, Threat Hunting.
* **Engineering:** KQL Development, API Integrations, Sensor Deployment.
* **Identity Security:** Active Directory Hardening, Kerberos Security, GPO Management.

  ⚙️ Deployment & Technical Implementation
1. Cloud Identity & Licensing
Tenant Setup: Provisioned a Microsoft 365 E5 Developer tenant to unlock the full XDR security suite.

License Management: Assigned Microsoft 365 E5 licenses to lab users via the Microsoft 365 Admin Center to enable Defender for Endpoint and Identity features.

2. Hybrid Infrastructure Integration
Azure Arc: Connected on-premises GOAD VMs (DC01, DC02, DC03) to Azure using Azure Arc for centralized management.

Azure Monitor Agent (AMA): Deployed the AMA to replace legacy agents, facilitating high-speed telemetry streaming to the cloud.

Data Collection Rules (DCR): Engineered specific DCRs to filter and stream granular Windows Event Logs (e.g., Security Events 4624, 4662) directly into Microsoft Sentinel.

3. Operationalizing XDR Sensors
Defender for Identity (MDI): Operationalized MDI sensors on all Domain Controllers; configured Directory Service Accounts (GMSA) to allow the sensors to parse Active Directory traffic.

Defender for Endpoint (MDE): Onboarded the GOAD workstation fleet and member servers (CASTELBLACK, BRAAVOS) using the Local Script method to verify immediate telemetry flow.

Sentinel Workspace: Architected the Log Analytics Workspace (LAW) and enabled the Microsoft Defender XDR Data Connector to aggregate cross-stack alerts.

---
## 📂 Repository Structure

* `/Architecture`: Lab diagrams and subnet mapping.
* `/Detections`: Custom KQL hunting queries and Analytics rules.
* `/Simulations`: Walkthroughs and logs of performed attacks.
* `/Automation`: Azure Logic App (JSON) templates for SOAR.
