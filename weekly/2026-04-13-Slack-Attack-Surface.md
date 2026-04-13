# Weekly – Slack Developer Phishing Campaign (Certificate + Malware Delivery)

**Date:** April 13, 2026  
**Severity:** High  
**Focus:** Social Engineering → Credential Access → Trust Subversion → Malware Execution  

---

## Overview

An active social engineering campaign is targeting open source developers via Slack communities (including ToDoGroup and related groups).  

Attackers are impersonating trusted community leaders to lure victims into a multi-stage attack involving:
- Phishing
- Credential harvesting
- Malicious certificate installation
- Malware execution

This campaign highlights a critical shift:  
**Attackers are targeting developer trust relationships and workflows rather than software vulnerabilities.**

---

##  Attack Chain Breakdown

1. **Initial Access (Slack)**
   - Impersonation of trusted community member  

2. **Phishing**
   - Fake Google Workspace link (`sites.google.com/...`)  

3. **Credential Harvesting**
   - Email + MFA code collection  

4. **Trust Subversion**
   - Fake “Google certificate” installation (MITM capability)  

5. **Malware Execution**
   - macOS: `gapi` binary from `2.26.97.61`  
   - Windows: malicious certificate install  

6. **Impact**
   - Credential theft  
   - Session hijacking  
   - Persistent interception of encrypted traffic  

---

##  MITRE ATT&CK Mapping

- T1566 – Phishing  
- T1204 – User Execution  
- T1553 – Subvert Trust Controls  
- T1556 – Modify Authentication Process  

Framework: :contentReference[oaicite:0]{index=0}

---

##  Data Components (DC Mapping with IDs)

### Identity / SaaS

- **DC0038 – Application Log Content**  
  Slack messages, user activity, impersonation indicators  

- **DC0002 – User Account Authentication**  
  Login attempts, MFA prompts  

- **DC0067 – Logon Session Creation**  
  Successful authentication and session establishment  

---

### Network

- **DC0103 – Active DNS**  
  DNS queries to lookalike domains (`sites.google.com/...`)  

- **DC0078 – Network Traffic Flow**  
  Outbound connections to suspicious IPs (e.g., `2.26.97.61`)  

- **DC0082 – Network Connection Creation**  
  Initial connection to attacker infrastructure  

- **DC0085 – Network Traffic Content** *(if available)*  
  HTTP/TLS inspection of phishing pages or downloads  

---

### Endpoint (Critical Visibility Layer)

- **DC0032 – Process Creation**  
  Execution of malicious binary (`gapi`)  

- **DC0064 – Command Execution**  
  Script execution (`curl`, `bash`, installers)  

- **DC0039 – File Creation**  
  Download of malicious payload  

- **DC0055 – File Access**  
  Execution or interaction with downloaded files  

- **DC0093 – Certificate Registration**  
  Detection of new or unauthorized root certificates  

- **DC0021 – OS API Execution**  
  System-level changes (certificate store modification)  

---

### Credential & Web Activity

- **DC0007 – Web Credential Usage**  
  User entering credentials into phishing site  

- **DC0006 – Web Credential Creation**  
  Creation of session tokens or auth artifacts  

---

### Malware / Threat Intelligence (Enrichment)

- **DC0011 – Malware Content**  
  Binary characteristics (`gapi`)  

- **DC0003 – Malware Metadata**  
  File hashes, origin IP, compile details  

---

##  Required Telemetry (SEC511 Focus)

To support the DCs above:

- Slack audit logs  
- DNS logs  
- Proxy / web logs  
- NetFlow / VPC Flow Logs  
- Endpoint EDR telemetry  
- OS certificate store monitoring  

---

##  Detection Opportunities

- Alert on **new root certificate installation**
- Detect **process execution from download/temp paths**
- Monitor **outbound connections to uncommon IPs**
- Flag **Slack messages containing authentication links**
- Detect **abnormal MFA activity**
- Correlate:
  - Slack message → link click → credential entry → process execution → outbound traffic  

---

##  Defensive Checklist

### Identity Verification
- [ ] Verify sender via separate trusted channel  
- [ ] Do not trust Slack display names alone  

### Link Safety
- [ ] Inspect URLs carefully  
- [ ] Manually navigate to trusted services  

### Credential Protection
- [ ] Never enter credentials from Slack links  
- [ ] Never share MFA codes  

### Certificate Security
- [ ] Never install certificates from websites  
- [ ] Monitor certificate store changes  

### Execution Safety
- [ ] Do not run untrusted scripts or binaries  
- [ ] Block `curl | bash` patterns  

---

##  Incident Response

- [ ] Disconnect affected system  
- [ ] Remove unauthorized certificates  
- [ ] Run EDR scans  
- [ ] Rotate passwords, SSH keys, API tokens  
- [ ] Revoke active sessions  
- [ ] Review logs for lateral movement  

---

##  Defensive Design Takeaways

- Slack = **initial access vector**  
- Certificates = **trust boundary attack**  
- Developers = **high-value identity targets**  

Visibility must span:
- Identity  
- Endpoint  
- Network  

 **No single data source detects this — correlation is required**

---

##  Key Insight

> Attackers are no longer just exploiting software —  
> they are exploiting **trust, identity, and developer workflows**.

---

##  Sources

- OpenSSF / Linux Foundation advisory  
- Slack community reports  
- MITRE ATT&CK Data Components :contentReference[oaicite:1]{index=1}  

---

##  SEC511 Alignment

- Threat → ATT&CK techniques  
- ATT&CK → Data Components (DCs)  
- DCs → Telemetry  
- Telemetry → Detection  

👉 **Full detection engineering pipeline demonstrated**
