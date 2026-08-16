# SOC Analysis Report & ISO/IEC 27001:2022 Assessment

**Analyst:** Adrián Steven Fajardo Liu  
**Date of Analysis:** August 14, 2026  
**Classification:** Confidential / Internal Project  
**Status:** Completed - Threat Confirmed  

---

## 1. Executive Summary

A technical investigation and security audit were performed on a suspicious email impersonating an official Microsoft service suspension alert. Forensic analysis confirms that this is an **active phishing campaign** aimed at credential harvesting through social engineering techniques and the exploitation of misconfigured infrastructure.

---

## 2. Email Analysis (Phishing)

### Social Engineering Indicators
* **Urgency & Coercion:** Use of visual alerts (`🚨 ALERTA: CUENTA SUSPENDIDA - VERIFICAR URGENTE 🚨`), flagged with *High Importance*, and threats of service cancellation if immediate action is not taken.
* **Spoofing / Unauthorized Sender:** While the interface pretends to originate from Microsoft, the actual sender address is `abraham.guzman24LGT@nube.unadmexico.mx` (an external educational subdomain unrelated to Microsoft).
* **Stylometric Deficiencies:** Recurrent spelling and punctuation errors (*politicas*, *actualizaciones importante*, *¿Tenes dudas?*).
* **False Legitimacy (Legal Filler):** Disproportionate citation of official government decrees (DOF) from 2020 related to the COVID-19 health emergency, used to confuse the user and fake authenticity.

---

## 3. Technical Infrastructure Analysis

### 3.1. URL Scanning (VirusTotal)
* **Analyzed URL:** `http://habilitar-email365x.hxstn.me/` $\rightarrow$ `https://habilitar-email365x.hxstn.me/`
* **Target IP:** `185.27.134.252`
* **HTTP Status:** `200 OK` (Active page)
* **Detection:** Flagged as **Phishing / Malicious** by security vendors (LevelBlue, Webroot).
* **Critical Tag:** Categorized with the **`password-input`** tag, technically validating the presence of an input form designed for credential harvesting.

### 3.2. Origin Domain Audit (MXToolbox)
The root domain analysis of the sender (`unadmexico.mx`) revealed severe security flaws:

| Detected Vulnerability | MXToolbox Result | Security Impact |
| :--- | :--- | :--- |
| **Blacklists** | `mta.unadmexico.mx` on Spamhaus ZEN | Compromised domain or active history of sending spam/malware. |
| **DMARC Policy** | `Quarantine/Reject policy not enabled` | Absence of strict enforcement rules to block spoofed emails (*spoofing*). |
| **SPF Record** | `More than one record found` | Invalidation of the SPF protocol, allowing unauthorized senders to dispatch emails on its behalf. |

---

## 4. ISO/IEC 27001:2022 Alignment & Audit

This incident was evaluated against the security controls of the **ISO/IEC 27001:2022** framework to identify normative gaps:

### Annex A Controls Mapping

1. **A.5.17 - Threat Intelligence:**
   * *Application:* Collection, extraction, and forensic analysis of IoCs (URLs, DNS records, IPs, and hashes) to understand the nature of the threat.
2. **A.8.7 - Protection against Malware & Malicious Code:**
   * *Application:* Identification and assessment of deception and social engineering vectors aimed at bypassing endpoint security controls.
3. **A.8.19 - Information Security Incident Management:**
   * *Application:* Documentation and formal logging of the incident lifecycle (Detection, Triage, Root Cause Analysis, and Containment Recommendations).

### Audit Findings (Non-Conformities / Gaps)

* **Deficiency in Email Authentication Controls (DMARC/SPF):** The lack of strict policies represents a systemic vulnerability that compromises the authenticity and integrity of corporate information.
* **Lack of Infrastructure Reputation Monitoring:** The persistence of SMTP servers on international blacklists reflects the absence of continuous monitoring and risk treatment processes for email assets.

---

## 5. Conclusions & Recommendations

1. **Verdict:** Confirmed high-risk **Credential Harvesting Phishing** campaign.
2. **Recommended Containment Actions:**
   * Block the domain `hxstn.me` and IP `185.27.134.252` at the perimeter/firewall level.
   * Notify the security team of the educational institution to mitigate unauthorized use of their institutional accounts.
   * Enforce DMARC policies (`p=reject`) and fix SPF records across email servers.
