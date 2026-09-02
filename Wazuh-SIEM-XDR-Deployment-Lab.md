# Wazuh SIEM/XDR Deployment & Integration (Ubuntu Manager & Windows Agent)

| Field | Value |
| :--- | :--- |
| **Author** | _(add your name)_ |
| **Date** | _(add the lab date)_ |
| **Wazuh Version** | 4.12.0 |
| **Document Type** | Deployment Guide / Lab Documentation |
| **Environment** | Lab (Home Lab) |
| **Status** | Completed |

## Summary

This repository documents the installation, configuration, and cryptographic pairing of a **Wazuh SIEM/XDR** platform in a lab environment. It covers the deployment of the central server on Ubuntu Linux and the enrollment of a Windows client endpoint for real-time security event monitoring.

---

## 🏗️ Lab Architecture

| Component | Operating System | IP Address | Network Role |
| :--- | :--- | :--- | :--- |
| Wazuh Manager / Dashboard / Indexer | Ubuntu Linux | `192.168.5.143` | Central SIEM Server |
| Wazuh Agent | Windows Host | `192.168.5.74` | Monitored Endpoint (ID: 001) |

---

## 🚀 Phase 1: Wazuh Manager Deployment & Configuration (Ubuntu)

### Step 1: Network Verification and Server IP Address

**Procedure / Commands:**
```bash
ifconfig
```

**Technical Explanation:** Identification of the active network interface (`wlp2s0`) and the assigned IP address (`192.168.5.143`). This IP is required to configure the Manager's listening ports for event ingestion (1514/1515 TCP/UDP) and access to the HTTPS Web console (443).

**Screenshot:**
![Network Verification](img/01_ubuntu_network_configuration.png)

> **PaC / Control Alignment:** ISO/IEC 27001:2022 A.8.20 (Network security) · CIS Controls 4 (Secure configuration of enterprise assets)

---

### Step 2: Importing the Wazuh Repository GPG Key

**Procedure / Commands:**
```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --dearmor -o /usr/share/keyrings/wazuh-archive-keyring.gpg
```

**Technical Explanation:** Cryptographic signature used to verify package integrity. The public GPG key ensures the installed binaries originate from Wazuh's official source and have not been tampered with in transit (prevention of supply-chain / MitM attacks).

**Screenshot:**
![GPG Key Import](img/02_wazuh_gpg_key_import.png)

> **PaC / Control Alignment:** ISO/IEC 27001:2022 A.8.24 (Use of cryptography) · NIST SP 800-53 SI-7 (Software, firmware, and information integrity)

---

### Step 3: Running the Automated Installation Wizard

**Procedure / Commands:**
```bash
curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh && sudo bash ./wazuh-install.sh -a -i
```

**Technical Explanation:** *All-in-One* deployment of the Wazuh architecture (v4.12.0). The script generates the local Certificate Authority (CA), inter-component TLS/SSL certificates (Indexer, Server, Dashboard, Filebeat), and configures indexed storage for security events.

**Screenshot:**
![Installer Execution](img/03_wazuh_installer_execution.png)

> **PaC / Control Alignment:** ISO/IEC 27001:2022 A.8.9 (Configuration management) · NIST SP 800-53 CM-2 (Baseline configuration)

---

### Step 4: Deployment Completion and Credential Generation

**Procedure:** Automatic output displayed by the installer.

**Technical Explanation:** Confirmation that the ecosystem has initialized. The admin URL (`https://<wazuh-dashboard-ip>:443`) and a high-entropy random password for the default administrative user (`admin`) are generated.

**Screenshot:**
![Installation Summary](img/04_wazuh_installation_summary_credentials.png)

> **PaC / Control Alignment:** ISO/IEC 27001:2022 A.5.17 (Authentication information) · NIST SP 800-53 IA-2 (User authentication)

---

### Step 5: Authenticating to the Web Console (Wazuh Dashboard)

**Procedure:** Navigate to `https://192.168.5.143` and enter the session's administrative credentials.

**Technical Explanation:** Sign-in to the centralized web interface. Communication is enforced over encrypted HTTPS/TLS (port 443) for secure SIEM/XDR management.

**Screenshot:**
![Dashboard Login](img/05_wazuh_dashboard_login.png)

> **PaC / Control Alignment:** ISO/IEC 27001:2022 A.8.26 (Application security requirements) · CIS Controls 6 (Access control management)

---

### Step 6: Verifying SIEM Status and Active Modules

**Procedure:** Navigate to the main *Overview* screen after authenticating.

**Technical Explanation:** Initial platform state prior to agent enrollment. Confirms activation of the analysis engines: *Configuration Assessment*, *Malware Detection*, *File Integrity Monitoring (FIM)*, *Threat Hunting*, *Vulnerability Detection*, *MITRE ATT&CK*, and compliance frameworks (PCI DSS, GDPR, HIPAA, NIST 800-53).

**Screenshot:**
![Initial Dashboard State](img/06_wazuh_dashboard_overview.png)

> **PaC / Control Alignment:** ISO/IEC 27001:2022 A.8.16 (Monitoring activities) · NIST SP 800-53 AU-6 (Audit record review, analysis, and reporting)

---

### Step 7: Manually Registering the Windows Agent on the Manager (CLI)

**Procedure / Commands:**
```bash
sudo /var/ossec/bin/manage_agents
```
- **Action:** `A` (Add an agent)
- **Name:** `WindowsHost`
- **IP Address:** `192.168.5.74`
- **Confirm:** `y`

**Technical Explanation:** Registration of a new endpoint in the Manager's local database (`client.keys`). A unique 3-digit identifier is assigned (ID: 001) and the target host's IP address is explicitly bound to it to prevent impersonation attacks (IP spoofing).

**Screenshot:**
![CLI Agent Registration](img/07_wazuh_cli_add_agent.png)

> **PaC / Control Alignment:** ISO/IEC 27001:2022 A.8.8 (Management of technical vulnerabilities / asset inventory) · CIS Controls 1 (Inventory and control of enterprise assets)

---

### Step 8: Extracting the Cryptographic Authentication Key

**Procedure / Commands:**
```bash
sudo /var/ossec/bin/manage_agents
```
- **Action:** `E` (Extract key)
- **Agent ID:** `001`

**Technical Explanation:** Generation of a Base64-encoded token containing the pre-shared symmetric key (PSK). This key is required by the Windows agent to authenticate the communication channel and encrypt the logs and alerts sent to the central server.

**Screenshot:**
![Key Extraction](img/08_wazuh_cli_extract_agent_key.png)

> **PaC / Control Alignment:** ISO/IEC 27001:2022 A.8.24 (Cryptographic key management) · NIST SP 800-53 SC-12 (Cryptographic key establishment)

---

## 💻 Phase 2: Wazuh Agent Deployment & Configuration (Windows)

### Step 9: Installing the Wazuh Agent on Windows

**Procedure:** Run the Wazuh agent MSI installer, accept the software license terms (GNU General Public License v2), and select *Install*.

**Technical Explanation:** Deployment of the agent binaries as a background resident service (`wazuh-agent`). This component is responsible for collecting Windows Event Viewer logs (Security, System, Application), system metrics, and kernel events.

**Screenshot:**
![Windows Agent Installer](img/09_windows_agent_installer.png)

> **PaC / Control Alignment:** ISO/IEC 27001:2022 A.8.8 (Management of technical vulnerabilities / asset inventory) · CIS Controls 10 (Malware defenses)

---

### Step 10: Opening the Agent Interface and Initial State

**Procedure:** Open the agent management GUI (*Wazuh Agent GUI*) as administrator.

**Technical Explanation:** Diagnostic of the initial, unpaired state. The interface confirms the authentication key has not been imported (*Auth key not imported*), the service is stopped (*Not Running*), and the default IP points to `0.0.0.0`.

**Screenshot:**
![Initial Agent State](img/10_windows_agent_initial_state.png)

> **PaC / Control Alignment:** ISO/IEC 27001:2022 A.8.20 (Network security)

---

### Step 11: Configuring the Wazuh Manager IP

**Procedure:** In the *Manager IP* field, enter the static IP address of the Ubuntu server where the Manager was installed (`192.168.5.143`).

**Technical Explanation:** Sets the telemetry traffic destination. Specifies the IP to which the agent will direct heartbeat packets and the encrypted event stream over UDP/TCP port 1514.

**Screenshot:**
![Manager IP Configuration](img/11_windows_agent_manager_ip_config.png)

> **PaC / Control Alignment:** ISO/IEC 27001:2022 A.8.20 (Network security) · NIST SP 800-53 CA-3 (Information exchange / system interconnections)

---

### Step 12: Importing the Cryptographic Authentication Key

**Procedure:** Paste the Base64 key extracted from the Manager into the *Authentication key* field, click *Save*, and confirm the pop-up dialog (*OK*).

**Technical Explanation:** Injects the shared secret into the target host's local `client.keys` file. The dialog confirms the binding for agent ID: `001`, Agent Name: `WindowsHost`, and IP Address: `192.168.5.74`. This enables the encrypted communication tunnel via AES/Blowfish.

**Screenshot:**
![Key Import](img/12_windows_agent_key_import.png)

> **PaC / Control Alignment:** ISO/IEC 27001:2022 A.8.24 (Cryptographic key management) · NIST SP 800-53 SC-12 (Cryptographic key establishment)

---

### Step 13: Confirming the Active Agent and Telemetry Reception

**Procedure:** Access the central Wazuh Dashboard panel and refresh the main *Overview* view.

**Technical Explanation:** Verification of successful end-to-end connectivity. The control panel shows *Active (1)* in the agents section and begins indexing the first medium-severity (Level 7-11) and low-severity (Level 0-6) alerts generated from Windows audit logs.

**Screenshot:**
![Active Agent on Dashboard](img/13_wazuh_dashboard_agent_active.png)

> **PaC / Control Alignment:** ISO/IEC 27001:2022 A.8.16 (Monitoring activities) · CIS Controls 8 (Audit log management)

---

## Suggested Next Steps

- Configure custom detection rules and *Active Response* on the Manager.
- Enable *File Integrity Monitoring (FIM)* on critical paths of the Windows agent.
- Document post-deployment hardening (rotating the `admin` password, closing unused ports, backing up the CA).

## References

* Wazuh Official Documentation — [https://documentation.wazuh.com](https://documentation.wazuh.com)
* ISO/IEC 27001:2022 — *Information security, cybersecurity and privacy protection — Information security management systems — Requirements.*
* NIST SP 800-53 Rev. 5 — *Security and Privacy Controls for Information Systems and Organizations.*
* CIS Controls v8 — Center for Internet Security.
