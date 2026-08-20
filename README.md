# 🛡️ Security Operations Center (SOC) Writeups

Welcome to my repository for incident analysis, threat triage, and cybersecurity lab writeups. This space documents practical methodologies, technical analysis of malicious artifacts, and SOC / Blue Team incident response workflows.

---

## 📁 Index of Writeups & Cases

| Document | Description / Focus | Category |
| :--- | :--- | :--- |
| 📄 [**Phishing OSINT Triage**](./Phishing-Osint-Triage.md) | Phishing email triage, IoC extraction, IP/Domain reputation checks, and OSINT investigation. | Threat Intelligence & OSINT |
| 📄 [**Phishing Sandbox Dynamic Analysis**](./Phishing-Sandbox-Dynamic-Analysis.md) | Dynamic analysis of suspicious attachments and links within an isolated environment (Sandbox). | Malware Analysis & Sandbox |
| 📄 [Callback Phishing (TOAD) & Document Triage](./SOC-Analysis-Report-TOAD-Phishing-ISO-IEC-27001-2022.md) | Analysis of TOAD callback campaign, SaaS infrastructure abuse, and static inspection of suspicious Word document. | Email Security & Threat Intelligence |

---

## 🛠️ Tools & Environments

- **OSINT & Reputation Analysis:** VirusTotal, MXToolbox, URLScan.io, AbuseIPDB, Whois.
- **Dynamic Analysis / Sandbox:** ANY.RUN, Hybrid Analysis, Kali Linux VM.
- **Networking & Traffic Analysis:** Wireshark, Tshark.
- **Reference Frameworks:** MITRE ATT&CK®, NIST SP 800-61 Rev. 2 (Computer Security Incident Handling Guide).

---

## 🚀 Report Structure

Each writeup in this repository follows a standardized analysis structure:
1. **Executive Summary:** High-level context of the threat or incident.
2. **Indicators of Compromise (IoCs):** IPs, Hashes (MD5/SHA256), URLs, and domains.
3. **Technical Analysis / Investigation Flow:** Step-by-step evidence and screenshots.
4. **MITRE ATT&CK Mapping:** Detected Tactics, Techniques, and Procedures (TTPs).
5. **Recommendations & Mitigation:** Suggested actions for remediation and defense.

---

📌 *This repository is continuously updated as I complete new security labs and investigations.*
