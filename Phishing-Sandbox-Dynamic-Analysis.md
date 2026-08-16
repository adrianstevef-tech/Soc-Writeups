# SOC Analysis Report & ISO/IEC 27001:2022 Assessment

**Analyst:** Adrián Steven Fajardo Liu  
**Date of Analysis:** August 16, 2026  
**Classification:** Confidential / Internal Project  
**Status:** Completed - Forensic & Dynamic Analysis Concluded  

---

## 1. Executive Summary

A technical investigation, dynamic analysis, and security audit were performed on an active Microsoft Outlook credential harvesting campaign. The investigation was conducted in an isolated sandbox environment (Ubuntu VM with Tor Browser routing). Forensic inspection of the network traffic and DOM structure confirmed that the threat actor utilized **Discord Webhooks** as an exfiltration vector, combined with client-side **IP fingerprinting/geofencing** and **CSS obfuscation** to evade automated security scanners.

---

## 2. Laboratory Setup & Sandbox Isolation

### Environmental Hardening Indicators
* **Operating System & Virtualization:** Ubuntu Linux Virtual Machine configured with NAT network isolation, disabled shared clipboard, and disabled drag-and-drop features.
* **Anonymization Routing:** All interactive web traffic was strictly routed through the **Tor Network** (`torbrowser-launcher`) to prevent operational IP leaks.
* **Anonymization Verification:** Egress traffic registered the Tor exit node IP `109.70.100.15` (Vienna, Austria), confirming zero leak of the analyst's origin IP address.

---

## 3. Technical Infrastructure Analysis

### 3.1. Landing Page & Interface Spoofing
* **Analyzed URL:** `http://habilitar-email365x.hxstn.me/?i=1` -> `https://habilitar-email365x.hxstn.me/?i=1`
* **Target Domain / IP:** `hxstn.me` (Free hosting provider / dynamic DNS)
* **Impersonated Target:** Microsoft Outlook (`login.microsoftonline.com`)
* **HTTP Status:** `200 OK` (Active credential harvesting landing page)
* **Visual Fidelity:** High-precision clone of official Microsoft authentication UI using external assets from `fonts.googleapis.com`.

### 3.2. Code Obfuscation & Validation (DOM Inspection)
The DOM structural audit revealed specific evasion and exfiltration techniques:

| Component | Technical Finding | Security Impact / Threat Purpose |
| :--- | :--- | :--- |
| **Exfiltration Vector** | `<form id="formularioDiscord">` | Utilizes Discord Webhooks for real-time exfiltration without backend DBs. |
| **CSS Obfuscation** | Class names like `DSFSDFSD2`, `XASDASD`, `TSDFXSE` | Randomized strings to bypass static signature detection engines. |
| **Input Validation** | `<span id="errorCorreo">` | JS validation restricting input strictly to `@hotmail`, `@outlook`, or `@live` domains. |

### 3.3. Dynamic Network Traffic Analysis (Fingerprinting & Recon)
Background HTTP traffic (`Fetch/XHR`) captured during page load confirmed automated client reconnaissance:

* **IP Discovery:** `GET https://api.ipify.org/?format=json` (Extracts visitor's public IP address).
* **Geofencing & Metadata:** `GET https://ipapi.co/109.70.100.15/json/` (Collects Country, City, Timezone, and ISP data).
* **Exfiltration Mechanism:** Outbound POST request triggered upon form submission towards a Discord Webhook API endpoint.

---

## 4. ISO/IEC 27001:2022 Alignment & Audit

This incident was evaluated against the security controls of the **ISO/IEC 27001:2022** framework to identify normative gaps and mitigation strategies:

### Annex A Controls Mapping

1. **A.5.17 - Threat Intelligence:**
   * *Application:* Collection, extraction, and forensic analysis of IoCs (URLs, dynamic DNS hosts, Webhook endpoints, and external API dependencies).
2. **A.8.7 - Protection against Malware & Malicious Code:**
   * *Application:* Analysis of anti-analysis tactics (CSS randomization, JS input filtering) to tune Secure Email Gateways (SEG) and EDR heuristics.
3. **A.8.12 - Data Leakage Prevention (DLP):**
   * *Application:* Detection and restriction of unauthorized outbound communications to third-party Webhook services (e.g., Discord API) from corporate endpoints.
4. **A.8.19 - Information Security Incident Management:**
   * *Application:* Formal documentation and triage of the incident lifecycle from sandbox isolation to root-cause investigation and IoC extraction.

### Audit Findings (Non-Conformities / Gaps)

* **Absence of Egress Webhook Filtering:** Lack of network controls blocking unauthorized API communication channels (e.g., Discord Webhooks) used for credential exfiltration.
* **Deficiency in Perimeter Domain Reputation Filtering:** Permitting outbound DNS requests to dynamic free-hosting domains (`hxstn.me`, `aeonfree.com`) commonly associated with disposable phishing campaigns.

---

## 5. Conclusions & Recommendations

1. **Verdict:** Confirmed active high-risk **Credential Harvesting Phishing** campaign utilizing Discord Webhook exfiltration and client fingerprinting.
2. **Recommended Containment Actions:**
   * Block domain `hxstn.me` and associated subdomains at perimeter firewalls, SWG, and DNS resolvers.
   * Restrict outbound HTTPS traffic to Discord Webhooks (`discord.com/api/webhooks/*`) across non-authorized enterprise endpoints.
   * Implement detection rules for authentication pages invoking client-side public IP discovery APIs (`ipify.org`, `ipapi.co`).
