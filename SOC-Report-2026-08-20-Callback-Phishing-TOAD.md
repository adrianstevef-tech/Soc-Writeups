# SOC Analysis Report: Callback Phishing (TOAD) & Attachment Inspection

| Field | Value |
| :--- | :--- |
| **Report ID** | SOC-2026-08-20-001 |
| **Analyst** | Adrián Steven Fajardo Liu |
| **Date of Analysis** | August 20, 2026 |
| **Incident Type** | Callback Phishing / TOAD (Telephone-Oriented Attack Delivery) |
| **Classification** | Confidential / Internal Project |
| **Severity** | Medium-High |
| **Status** | Completed – Static Analysis & Infrastructure Triage Concluded |

---

## 1. Executive Summary

A security investigation and email triage were conducted on a fraudulent transaction notification impersonating **McAfee™ Premium Protection**. The attack leverages **Telephone-Oriented Attack Delivery (TOAD)**, bypassing traditional Secure Email Gateways (SEG) by abusing a legitimate enterprise SaaS domain (`parentsquare.com`) and attaching an antivirus-undetected `.docx` file (0/65 on VirusTotal). The objective of the threat actor is out-of-band social engineering via voice phishing (vishing) through a malicious support number (`+1 (805) 946 4976`).

---

## 2. Indicators of Compromise (IOC Summary)

| Type | Value | Context |
| :--- | :--- | :--- |
| Sender email | `E1w084945hnHL-8fh04oe0ER2084945-244W@charlotte084945.parentsquare.com` | Abused multi-tenant subdomain used to deliver the lure |
| Abused subdomain | `charlotte084945.parentsquare.com` | Compromised/trial tenant leveraged for mass mail distribution |
| Root domain (legitimate, abused) | `parentsquare.com` | High-reputation educational SaaS platform; not itself malicious |
| Vishing phone number | `+1 (805) 946 4976` | Callback line used to socially engineer victims into installing a RAT |
| Malicious lure attachment | `M89FT17BRZ-073447.docx` | 0/65 on VirusTotal; static invoice lure |
| MD5 | `d69d1d0a4d68915369dd1dfc9f73dafa` | Attachment hash |
| SHA-256 | `80f4c99236970fa57b4a77f3ebff2652e6dd5fbb89f1a73094f41e625b856faf` | Attachment hash |
| Associated executable | `pjkbus.exe` | Correlated secondary artifact from sandbox execution history |

---

## 3. Incident Timeline & Root Cause Analysis (RCA)

### 3.1 Timeline

| Date / Time | Event | Phase |
| :--- | :--- | :--- |
| _(not specified in source material)_ | Fraudulent invoice email received by the target, impersonating McAfee Premium Protection | Delivery |
| 2026-08-20 | SOC analyst begins formal investigation and email triage | Detection / Triage |
| 2026-08-20 | Sender/domain analysis: SPF/DKIM/DMARC found to pass due to abuse of legitimate `parentsquare.com` infrastructure | Analysis |
| 2026-08-20 | Attachment `M89FT17BRZ-073447.docx` submitted to VirusTotal — 0/65 detection, `calls-wmi` behavioral tag identified | Static / Behavioral Analysis |
| 2026-08-20 | Correlation with secondary artifact `pjkbus.exe` from historical sandbox executions | Threat Correlation |
| 2026-08-20 | Containment and takedown recommendations issued | Containment / Recommendation |

*Note: the original delivery timestamp was not captured — add it from the email header (`Received:` / `Date:` fields) to complete the timeline.*

### 3.2 Root Cause Analysis (RCA)

Unlike a typical spoofing case, this incident succeeded **despite valid SPF/DKIM/DMARC authentication**, because the abuse occurred at the tenant level of a legitimate, high-reputation SaaS platform (`parentsquare.com`, registered since 2010, hosted on Google Workspace/AWS DNS). The root cause is twofold:

1. **Lack of subdomain-level reputation controls:** perimeter email filters trust the parent domain `parentsquare.com` without granular inspection of anomalous, randomly-generated tenant subdomains (e.g., `charlotte084945`).
2. **Absence of an out-of-band (vishing) detection layer:** by removing all hyperlinks and executable payloads from the email body, the attacker evades URL sandboxing and signature-based AV entirely, shifting the attack to a phone-based social-engineering channel that most email security controls do not monitor.

---

## 4. Threat Vector & Email Infrastructure Analysis

### 4.1 Sender & Domain Spoofing Analysis
* **Sender Address:** `E1w084945hnHL-8fh04oe0ER2084945-244W@charlotte084945.parentsquare.com`
* **Root Domain:** `parentsquare.com` (Legitimate Educational Communication Platform)
* **Domain Age (WHOIS):** Registered on September 14, 2010 (16-year high-reputation domain).
* **DNS & Mail Infrastructure:** AWS DNS (`awsdns`) and Google Workspace Mail Servers (`aspmx.l.google.com`).
* **Authentication Status (SPF/DKIM/DMARC):** Passed (valid delivery due to abuse of legitimate tenant infrastructure).
* **Specific Subdomain Target:** `charlotte084945` (indicates compromised tenant/trial account used for mass mail distribution).

### 4.2 Social Engineering & Evasion Tactics

| Attack Component | Indicator | Threat Purpose / Evasion Mechanism |
| :--- | :--- | :--- |
| **Primary Vector** | Callback Support Line (`+1 (805) 946 4976`) | Forces victims to initiate telephone contact (TOAD) to download Remote Access Trojans (RATs). |
| **False Invoice Charge** | USD $247.88 | Creates passive financial panic prompting immediate dispute action without urgent language. |
| **Link Absence** | Zero HTTP/HTTPS hyperlinks in body | Bypasses automated URL sandbox detonation engines. |
| **Infrastructure Abuse** | Google MX via ParentSquare | Leverages high-reputation IP space to prevent domain blocklisting. |

---

## 5. Static Document Forensic Analysis

### 5.1 File Identifiers
* **Filename:** `M89FT17BRZ-073447.docx`
* **File Size:** 26.45 KB (27082 bytes)
* **File Type:** Office Open XML Document (Microsoft Word 2007+)
* **MD5:** `d69d1d0a4d68915369dd1dfc9f73dafa`
* **SHA-256:** `80f4c99236970fa57b4a77f3ebff2652e6dd5fbb89f1a73094f41e625b856faf`

### 5.2 Static Threat Intelligence (VirusTotal)
* **Detection Score:** **0/65 (Undetected)** across all commercial AV engines.
* **Evasion Strategy:** The file functions primarily as a static invoice lure lacking active executable payloads, neutralizing static signature-based detection.
* **Behavioral Tags:** `calls-wmi` (indicates historical sandbox interactions with Windows Management Instrumentation for environment profiling).
* **Associated Artefacts:** Correlated execution history with secondary executable `pjkbus.exe`.

### 5.3 Suggested Additional Evidence (Screenshots)

This report currently has no visual evidence attached. To align with report standards, consider capturing:

| Evidence | Suggested Command / Source |
| :--- | :--- |
| VirusTotal detection page (0/65) | Screenshot of the VT report for the SHA-256 hash |
| WHOIS lookup of `parentsquare.com` | `whois parentsquare.com` |
| File metadata / type confirmation | `file M89FT17BRZ-073447.docx` or `exiftool M89FT17BRZ-073447.docx` |
| Hash verification | `sha256sum M89FT17BRZ-073447.docx` |
| Reverse phone number lookup | OSINT lookup on `+1 (805) 946 4976` (e.g., Whitepages, community abuse databases) |

---

## 6. ISO/IEC 27001:2022 Controls Alignment

### 6.1 Annex A Controls Mapping

1. **A.5.17 – Threat Intelligence:** Extraction of out-of-band IoCs (fraudulent phone numbers, abused subdomains, document hashes) for organizational threat feeds.
2. **A.8.7 – Protection against Malware:** Validation that signature-based controls fail against TOAD lures, reinforcing the need for heuristic and behavioral analysis.
3. **A.8.12 – Data Leakage Prevention (DLP) & Boundary Defense:** Hardening perimeter filters against unauthorized tenant subdomains from legitimate multi-tenant SaaS providers.
4. **A.8.23 – Web & Email Filtering:** Implementing strict inspection rules for emails containing financial lure keywords combined with non-standard tenant subdomains.

### 6.2 Policy as Code (PaC) – Control Mapping

Example mapping the subdomain-filtering finding (A.8.23) as a reusable, machine-readable policy check:

```yaml
# posture/controls/A8.23-tenant-subdomain-filtering.yaml
control_id: A.8.23
control_name: "Web & Email Filtering"
related_controls: ["A.8.12", "A.5.17"]
policy_id: "saas-tenant-subdomain-anomaly"
requirement: "Flag/block emails from randomly-generated tenant subdomains of trusted SaaS parent domains"
check:
  type: email_header_pattern
  field: "sender_domain"
  parent_domain: "parentsquare.com"
  flag_pattern: "^[a-z]+[0-9]{6}\\.parentsquare\\.com$"
status: "non_compliant"
evidence: "SOC-2026-08-20-001 - sender charlotte084945.parentsquare.com passed SPF/DKIM/DMARC but matched anomalous subdomain pattern"
severity: "high"
owner: "Email Security Team"
remediation: "Deploy subdomain-reputation/anomaly rule in SEG; alert on first-seen tenant subdomains"
```

### 6.3 Audit Findings (Non-Conformities / Gaps)

* **No subdomain-level reputation scoring:** perimeter defenses evaluate domain trust at the root level only, missing abuse of individual tenants within legitimate multi-tenant SaaS platforms.
* **No vishing/telephony threat intelligence feed:** the organization lacks a mechanism to flag or block known fraud-associated phone numbers referenced in inbound email.
* **Over-reliance on signature-based AV:** a 0/65 detection score on a credential-harvesting lure confirms static AV alone is insufficient without behavioral/heuristic layers.

---

## 7. Containment & Remediation Actions

1. **Subdomain Granular Block:** Block incoming traffic from `*.charlotte084945.parentsquare.com` and flag subdomains from `parentsquare.com` containing random alphanumeric prefixes.
2. **Abuse Reporting (Takedown):** Submit abuse notification to GoDaddy Registrar (`abuse@godaddy.com`) and ParentSquare Security Operations with email headers and sub-tenant details.
3. **Telecom / Vishing Defense:** Add telephone number `+1 (805) 946 4976` to corporate telephony blocklists and alert internal SOC voice-fraud monitoring systems.
4. **User Awareness (Security Training):** Publish an internal advisory regarding Callback Phishing (TOAD) campaigns impersonating subscription renewals (McAfee, GeekSquad, PayPal) requesting telephone contact.

---

## 8. References

* ISO/IEC 27001:2022 — *Information security, cybersecurity and privacy protection — Information security management systems — Requirements.*
* VirusTotal — [https://www.virustotal.com](https://www.virustotal.com)
* WHOIS domain lookup — [https://www.whois.com](https://www.whois.com)
* Open Policy Agent (OPA) / Rego documentation — [https://www.openpolicyagent.org/docs](https://www.openpolicyagent.org/docs)
* CISA — Advisory on Telephone-Oriented Attack Delivery (TOAD) / Callback Phishing techniques.
