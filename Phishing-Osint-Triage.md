# SOC Analysis Report & ISO/IEC 27001:2022 Assessment

| Field | Value |
| :--- | :--- |
| **Report ID** | SOC-2026-08-14-001 |
| **Analyst** | Adrián Steven Fajardo Liu |
| **Date of Analysis** | August 14, 2026 |
| **Incident Type** | Phishing / Credential Harvesting |
| **Classification** | Confidential / Internal Project |
| **Severity** | High |
| **Status** | Completed – Threat Confirmed |

> **Template note:** This structure (metadata table → Executive Summary → IOC Table → Timeline/RCA → Analysis → ISO 27001 & PaC → Findings → Conclusions → References) is meant to be reused as-is across future reports to keep formatting consistent.

---

## 1. Executive Summary

A technical investigation and security audit were performed on a suspicious email impersonating an official Microsoft service suspension alert. Forensic analysis confirms that this is an **active phishing campaign** aimed at credential harvesting through social engineering techniques and the exploitation of misconfigured infrastructure belonging to a third-party educational domain.

---

## 2. Indicators of Compromise (IOC Summary)

| Type | Value | Context |
| :--- | :--- | :--- |
| Sender email | `abraham.guzman24LGT@nube.unadmexico.mx` | Spoofed/abused institutional address used to deliver the phishing email |
| Phishing URL | `https://habilitar-email365x.hxstn.me/` | Credential-harvesting landing page (tagged `password-input` by VirusTotal) |
| Malicious domain | `hxstn.me` | Domain hosting the phishing infrastructure |
| Hosting IP | `185.27.134.252` | Server IP serving the phishing page (HTTP 200, `openresty`) |
| Abused legitimate domain | `unadmexico.mx` | Educational domain with weak DMARC/SPF, exploited to lend false legitimacy |

*Recommendation for future reports: export this table as CSV/STIX alongside the report so it can be ingested directly into a SIEM or threat intel platform.*

---

## 3. Incident Timeline & Root Cause Analysis (RCA)

### 3.1 Timeline

| Date / Time (approx.) | Event | Phase |
| :--- | :--- | :--- |
| 2026-07-31 01:17 UTC | Phishing URL `habilitar-email365x.hxstn.me` first indexed/scanned on VirusTotal — infrastructure already active | Pre-incident (Threat Staging) |
| 2026-08-11, 12:47 (local) | Phishing email received by the victim, spoofing Microsoft, flagged *High Importance* | Delivery |
| 2026-08-14 | SOC analyst begins formal investigation | Detection / Triage |
| 2026-08-14 | URL re-scanned on VirusTotal — confirmed Phishing/Malicious (LevelBlue, Webroot) | Analysis |
| 2026-08-14 | Domain audit of `unadmexico.mx` via MXToolbox — DMARC/SPF failures and Spamhaus ZEN blacklist identified | Root Cause Analysis |
| 2026-08-14 | Containment recommendations issued (block domain/IP, notify institution, enforce DMARC) | Containment / Recommendation |

*Note: timestamps above mix local time and UTC as captured from the original sources. Standardize all timestamps to UTC in future reports.*

### 3.2 Root Cause Analysis (RCA)

The most probable root cause is not an isolated failure but a **sustained, systemic misconfiguration** of the `unadmexico.mx` mail infrastructure:

- An **invalid SPF record** (multiple records found) that prevents receiving servers from reliably validating authorized senders.
- **No DMARC enforcement policy** (`quarantine`/`reject`), meaning spoofed or abused messages are not blocked even when authentication fails.
- **Historical presence on Spamhaus ZEN**, indicating the mail server had already been abused or compromised for spam/malware delivery prior to this incident.

Together, these gaps allowed a threat actor to send (or spoof) a message appearing to originate from a legitimate institutional account, bypassing standard email-authentication controls.

---

## 4. Email Analysis (Phishing)

**Evidence:** `evidence/01-phishing-email.png`

### Social Engineering Indicators
* **Urgency & Coercion:** Use of visual alerts (`🚨 ALERTA: CUENTA SUSPENDIDA - VERIFICAR URGENTE 🚨`), flagged with *High Importance*, and threats of service cancellation if immediate action is not taken.
* **Spoofing / Unauthorized Sender:** While the interface pretends to originate from Microsoft, the actual sender address is `abraham.guzman24LGT@nube.unadmexico.mx` (an external educational subdomain unrelated to Microsoft).
* **Stylometric Deficiencies:** Recurrent spelling and punctuation errors (*politicas*, *actualizaciones importante*, *¿Tenes dudas?*).
* **False Legitimacy (Legal Filler):** Disproportionate citation of official government decrees (DOF) from 2020 related to the COVID-19 health emergency, used to confuse the user and fake authenticity.

---

## 5. Technical Infrastructure Analysis

### 5.1 URL Scanning (VirusTotal)

**Evidence:** `evidence/02-virustotal-detection.png`, `evidence/03-http-response.png`

* **Analyzed URL:** `http://habilitar-email365x.hxstn.me/` → `https://habilitar-email365x.hxstn.me/`
* **Target IP:** `185.27.134.252`
* **HTTP Status:** `200 OK` (Active page)
* **Detection:** Flagged as **Phishing / Malicious** by security vendors (LevelBlue, Webroot).
* **Critical Tag:** Categorized with the **`password-input`** tag, technically validating the presence of an input form designed for credential harvesting.

### 5.2 Origin Domain Audit (MXToolbox)

**Evidence:** `evidence/04-mxtoolbox-audit.png`

The root domain analysis of the sender (`unadmexico.mx`) revealed severe security flaws:

| Detected Vulnerability | MXToolbox Result | Security Impact |
| :--- | :--- | :--- |
| **Blacklists** | `mta.unadmexico.mx` on Spamhaus ZEN | Compromised domain or active history of sending spam/malware. |
| **DMARC Policy** | `Quarantine/Reject policy not enabled` | Absence of strict enforcement rules to block spoofed emails (*spoofing*). |
| **SPF Record** | `More than one record found` | Invalidation of the SPF protocol, allowing unauthorized senders to dispatch emails on its behalf. |

### 5.3 Suggested Additional OSINT Evidence (Kali Linux)

To strengthen future reports with hands-on, terminal-based evidence (recommended for portfolio/interview purposes), the following tools can be run against these IOCs in a Kali VM and captured as screenshots:

| Tool | Purpose |
| :--- | :--- |
| `whois hxstn.me` / `whois 185.27.134.252` | Domain/IP registration, registrar, creation date |
| `dig` / `nslookup` | DNS resolution, MX/SPF/DMARC TXT record inspection |
| `theHarvester -d unadmexico.mx` | Passive recon of emails/subdomains tied to the abused domain |
| `dnsenum` / `dnsrecon` | Subdomain enumeration and DNS zone weaknesses |
| `amass enum -d unadmexico.mx` | Attack-surface / subdomain mapping |
| `curl -I` on the phishing URL | Manual HTTP header capture (cross-check vs. VirusTotal) |

*(Not executed for this report — flagged here as a follow-up action item.)*

---

## 6. ISO/IEC 27001:2022 Alignment & Audit

This incident was evaluated against the security controls of the **ISO/IEC 27001:2022** framework to identify normative gaps.

### 6.1 Annex A Controls Mapping

1. **A.5.17 – Threat Intelligence:** Collection, extraction, and forensic analysis of IoCs (URLs, DNS records, IPs, and hashes) to understand the nature of the threat.
2. **A.8.7 – Protection against Malware & Malicious Code:** Identification and assessment of deception and social engineering vectors aimed at bypassing endpoint security controls.
3. **A.8.19 – Information Security Incident Management:** Documentation and formal logging of the incident lifecycle (Detection, Triage, Root Cause Analysis, and Containment Recommendations).

### 6.2 Policy as Code (PaC) – Control Mapping

Instead of describing controls only in prose, they can be expressed as structured, machine-readable policy definitions. This enables automated compliance checks (e.g., with **Open Policy Agent / Rego**, Chef InSpec, or custom CI/CD scripts) instead of manual audits, and gives a clear audit trail per control.

Example — mapping the DMARC finding from this incident as code:

```yaml
# posture/controls/A5.17-email-authentication.yaml
control_id: A.5.17
control_name: "Threat Intelligence"
related_controls: ["A.8.19"]
policy_id: "email-security-dmarc"
requirement: "DMARC policy must be set to p=reject or p=quarantine"
check:
  type: dns_txt_record
  record: "_dmarc.{{domain}}"
  expected_pattern: "p=(reject|quarantine)"
target:
  domain: "unadmexico.mx"
status: "non_compliant"
evidence: "MXToolbox scan 2026-08-14 - 'DMARC Quarantine/Reject policy not enabled'"
severity: "high"
owner: "Email Security Team"
remediation: "Publish a DMARC record with p=reject within 30 days"
```

This same pattern can be replicated for the SPF and blacklist findings, giving you a small library of reusable "posture-as-code" checks across future reports.

### 6.3 Audit Findings (Non-Conformities / Gaps)

* **Deficiency in Email Authentication Controls (DMARC/SPF):** The lack of strict policies represents a systemic vulnerability that compromises the authenticity and integrity of corporate information.
* **Lack of Infrastructure Reputation Monitoring:** The persistence of SMTP servers on international blacklists reflects the absence of continuous monitoring and risk treatment processes for email assets.

---

## 7. Conclusions & Recommendations

1. **Verdict:** Confirmed high-risk **Credential Harvesting Phishing** campaign.
2. **Recommended Containment Actions:**
   * Block the domain `hxstn.me` and IP `185.27.134.252` at the perimeter/firewall level.
   * Notify the security team of the educational institution to mitigate unauthorized use of their institutional accounts.
   * Enforce DMARC policies (`p=reject`) and fix SPF records across email servers.
3. **Follow-up items:** Capture OSINT/CLI evidence (Section 5.3) and formalize the PaC checks (Section 6.2) into a reusable policy repository.

---

## 8. References

* ISO/IEC 27001:2022 — *Information security, cybersecurity and privacy protection — Information security management systems — Requirements.*
* VirusTotal — [https://www.virustotal.com](https://www.virustotal.com)
* MXToolbox — [https://mxtoolbox.com](https://mxtoolbox.com)
* Spamhaus ZEN Blocklist — [https://www.spamhaus.org](https://www.spamhaus.org)
* Open Policy Agent (OPA) / Rego documentation — [https://www.openpolicyagent.org/docs](https://www.openpolicyagent.org/docs)
