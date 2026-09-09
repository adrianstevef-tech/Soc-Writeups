# SOC Incident Analysis: Amazon Brand Impersonation & Callback Phishing Campaign

---

| Field | Value |
| :--- | :--- |
| **Report ID** | SOC-2026-08-28-002 |
| **Analyst** | Adrián Steven Fajardo Liu |
| **Date of Analysis** | August 28, 2026 |
| **Incident Type** | Callback Phishing / Vishing Scam / Impersonation |
| **Classification** | Confidential / Threat Intelligence Analysis |
| **Severity** | High |
| **Status** | Completed – Threat Confirmed |

> **Template Note:** This documentation follows standard SOC L1/L2 incident reporting guidelines (Metadata → Executive Summary → Technical Evidence → Threat OSINT → IOCs → Security Mapping → Remediation → References) optimized for direct rendering on GitHub.

---

## 1. Executive Summary

A forensic analysis was conducted on an unsolicited email and attached PDF document impersonating an official **Amazon Order Confirmation**. The campaign employs a **Callback Phishing (Vishing Hybrid)** attack model designed to bypass automated security filters.

Rather than embedding malicious URLs, the threat actor utilized an unauthorized $1,199.99 USD order confirmation for an *iPhone 17 Pro Max* to induce panic. The victim is directed to call fraudulent "Customer Support" telephone numbers controlled by the attacker to cancel the charge, creating an entry point for credential harvesting, social engineering, and remote access software execution.

---

## 2. Incident Overview & Attack Vector

| Attribute | Details |
| :--- | :--- |
| **Impersonated Brand** | Amazon Inc. |
| **Attacker Name / Sender** | `EDWARD GOMEZ` |
| **Target Address** | `adriansteve99@hotmail.com` |
| **Claimed Transaction** | $1,199.99 USD (Apple iPhone 17 Pro Max 512GB) |
| **Primary Vector** | Email Delivery → PDF/Notification Attachment → Vishing (Voice Phishing) |
| **Fraud Hotline Numbers** | `+1 (656) 556-2958` / `+1 (656) 556-5065` |
| **Attacker Goal** | Remote Access Trojan (RAT) Installation / Financial Fraud / Credential Theft |

---

## 3. Technical Evidence & Forensic Investigation

### Step 1: Initial Email Delivery & Inbox Inspection

* **Procedure:** Inspection of the email message header and notification body in the recipient's mail client.
* **Analysis:** The message originates from a display name labeled `EDWARD GOMEZ` with the subject referencing an order document (`ORD-TUDGX0A21UHE1QCZX7Q`). The body contains short text urging the recipient to review the attached document and contact support at `+1 (656) 556-5065`.
* **Key Finding:** The sender identity shows no affiliation with official Amazon corporate communication infrastructure.

![Inbox View & Sender Inspection]([img/01_phishing_email_inbox_header.png](https://github.com/adrianstevef-tech/Soc-Writeups/blob/849462db212ce987fcd678b2d3cc75d94d3e57e5/a49ce89f-ec5c-466e-bf05-bdd1987b1766.png)

---

### Step 2: Attached Invoice Analysis (Social Engineering Payload)

* **Procedure:** Extraction and static analysis of the embedded PDF invoice image (`Date: August 28, 2026`).
* **Analysis:**
  * **Visual Crafting:** Uses official Amazon branding logos, order tracking identifiers (`H4-2026_430437EMUO_YN` and `IT-1394381551259223342`), and high-value item details (*iPhone 17 Pro Max - Desert Titanium Color*) to maximize psychological urgency.
  * **Linguistic Anomalies:** The document contains grammatical errors typical of offshore scam templates (*"The transaction amount of $1199.99 USD is complete successfully"*, *"Give as call at:+1 (656) 556-5065"*).
  * **Callback Call-to-Action:** Prominently features two distinct support phone numbers (`+1 (656) 556-2958` and `+1 (656) 556-5065`).

![Fake Amazon PDF Invoice](img/02_amazon_fake_invoice_pdf.png)

---

### Step 3: Telephone Number OSINT & Telecommunication Lookup

* **Procedure:** Querying the primary callback number (`+1 (656) 556-5065`) on carrier lookup and threat intelligence platforms.
* **Analysis:**
  * **Carrier:** `TELIMIZE, LLC` (VoIP / Virtual Provider).
  * **Geographic Origin:** Florida, United States.
  * **Number Type:** Non-disposable VoIP line used to host automated interactive voice response (IVR) or call-center scams.

![Phone Number Carrier OSINT](img/03_phone_number_osint_lookup.png)

---

### Step 4: Email Domain Infrastructure & Security Assessment

* **Procedure:** Evaluating email domain health, MX records, and DMARC/SPF policy alignment using MXToolbox.
* **Analysis:** The domain health check identifies critical policy misconfigurations across public mail services, including missing `DMARC Quarantine/Reject` enforcement and rDNS banner mismatches. Threat actors exploit weak authentication policies on third-party relays to deliver impersonation lures into target inboxes.

![Domain Health Assessment](img/04_domain_health_mxtoolbox.png)

---

### Step 5: Domain WHOIS Record Verification

* **Procedure:** Querying WHOIS registration data for the infrastructure used during the delivery chain.
* **Analysis:** Verification confirms legitimate brand domain records vs. spoofed/compromised senders. The lookup validates registration dates, name servers, and abuse contact channels (`abusecomplaints@markmonitor.com`).

![Domain WHOIS Lookup](img/05_whois_domain_lookup.png)

---

## 4. Indicators of Compromise (IOCs)

| IOC Type | Indicator Value | Threat Context |
| :--- | :--- | :--- |
| **Sender Name** | `EDWARD GOMEZ` | Fraudulent sender account |
| **Fraud Phone #1** | `+1 (656) 556-2958` | Fraudulent customer support hotline |
| **Fraud Phone #2** | `+1 (656) 556-5065` | Fraudulent customer support hotline (TELIMIZE, LLC) |
| **Order Ref #1** | `H4-2026_430437EMUO_YN` | Fake Amazon Order Identifier |
| **Order Ref #2** | `ORD-TUDGX0A21UHE1QCZX7Q` | Fake Order Document ID |
| **Transaction ID** | `D8-1394381551259223342` | Fake Payment Reference |

---

## 5. Security Framework Alignment

### MITRE ATT&CK Mapping

| Tactic | Technique ID | Technique Name | Description |
| :--- | :--- | :--- | :--- |
| **Initial Access** | `T1566.001` | Phishing: Spearphishing Attachment | Delivery of malicious PDF invoice lure |
| **Reconnaissance** | `T1598.003` | Phishing for Information: Social Engineering | Use of fake high-value transaction to prompt contact |
| **Execution** | `T1204.001` | User Execution: Malicious Link / Call | Coercing victim into dialing scam support hotline |

---

### ISO/IEC 27001:2022 Control Alignment

* **A.5.10 (Acceptable use of information and assets):** Establishes guidelines to prevent response to unverified financial communications.
* **A.8.12 (Data leakage prevention):** Prevents users from disclosing credentials or banking details via unauthorized telephone calls.
* **A.8.16 (Monitoring activities):** Ingestion and correlation of email gateway logs to detect domain spoofing.
* **A.8.23 (Web filtering / Email security):** Mandatory enforcement of `DMARC p=reject` and SPF validation on email security gateways.

---

## 6. Root Cause Analysis & Mitigation Plan

### Root Cause
Threat actors leveraged weak email alignment and VoIP phone infrastructure to bypass traditional signature-based email filters. By avoiding embedded hyperlinks, the attack successfully reached the user's inbox relying entirely on social engineering (fear of financial loss).

### Recommended SOC Mitigation Actions

1. **Email Gateway Rules:**
   * Create keyword regex filters blocking messages combining phrases like *"Amazon Order Complete"*, *"Transaction ID"*, and non-standard support phone formats.
   * Enforce strict DMARC and DKIM validation at the mail gateway.

2. **Telecom & Endpoint Defenses:**
   * Submit identified VoIP phone numbers (`+1 656-556-5065` / `+1 656-556-2958`) to FTC / carrier abuse databases for takedown.
   * Restrict remote desktop administration tools (e.g., AnyDesk, TeamViewer, Quick Assist) on endpoints using AppLocker / Software Restriction Policies.

3. **Security Awareness:**
   * Educate employees/users to verify billing claims exclusively through official web portals (`amazon.com/orders`), never utilizing telephone numbers listed inside unverified email notifications.

---

## 7. References

* **Amazon Security Guidance:** *Recognizing Phishing Emails and Fraudulent Phone Calls*
* **MITRE ATT&CK Framework:** [https://attack.mitre.org](https://attack.mitre.org)
* **ISO/IEC 27001:2022:** *Information security, cybersecurity and privacy protection — Security controls.*
