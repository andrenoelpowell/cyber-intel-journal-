# SEC511 Weekly Cyber Intel Journal
## Week of March 30, 2026
### Topic: Trivy Supply Chain Attack and CI/CD Secret Exposure

---

## 1. Executive Summary

This week I analyzed the March 2026 **Trivy supply chain attack**, in which attackers abused compromised credentials and repository automation to distribute malicious Trivy artifacts and GitHub Actions references.

The threat actor:
- Published a malicious `trivy` v0.69.4 release
- Force-pushed **76 of 77 tags** in `trivy-action`
- Replaced all `setup-trivy` tags with malicious commits
- Later published malicious Docker images (`v0.69.5`, `v0.69.6`)

The malware executed **before legitimate scanning logic**, allowing CI/CD jobs to appear successful while silently extracting secrets from runner environments.

This attack demonstrates a critical reality:
> **Trusted security tools can become the attack vector.**

---

## 2. Why This Event Matters

This incident is significant because it targeted:
- **CI/CD pipelines (not endpoints)**
- **Trusted open-source tooling**
- **Software distribution channels**

Instead of attacking a single organization, the adversary leveraged:
- GitHub Actions
- Release pipelines
- Mutable version tags

This allowed **scalable compromise across downstream users**.

### Key lesson:
Security visibility must extend to:
- CI/CD runners
- Build pipelines
- Release systems
- Automation identities
- Secrets handling

---

## 3. Attack Overview

### Timeline

- **Late Feb 2026** – GitHub Actions misconfiguration → token theft  
- **March 1** – Credential rotation (incomplete)  
- **March 19** – Malicious releases + tag hijacking  
- **March 20** – Containment and remediation  
- **March 22** – Malicious Docker images discovered  

---

### Exposure Window Analysis

- **trivy binary**: ~3 hours  
- **trivy-action**: ~12 hours  
- **setup-trivy**: ~4 hours  
- **Docker images**: ~10 hours :contentReference[oaicite:0]{index=0}  

This highlights a key detection requirement:

> Defenders must perform **time-scoped retrospective analysis** of CI/CD activity.

---

### Attack Execution Details

The malicious payload executed **before legitimate Trivy scanning** and performed:

- Memory dumping via `/proc/<pid>/mem`
- Enumeration of 50+ credential locations:
  - AWS, GCP, Azure credentials
  - SSH keys
  - Kubernetes tokens
  - Docker configs
  - `.env` files
- Encryption using **AES-256-CBC + RSA-4096**
- Exfiltration to attacker infrastructure
- Fallback exfiltration via GitHub repo:
  - `tpcp-docs`

This represents a **full credential harvesting operation inside CI/CD runners**. :contentReference[oaicite:1]{index=1}  

---

### Why the Attack Was Effective

- Mutable version tags were trusted (`@v0.x`)
- CI/CD runners had access to high-value secrets
- Pipeline success masked malicious activity
- Credential rotation was incomplete (non-atomic)

---

## 4. MITRE ATT&CK Mapping

### Primary Technique
**T1195.001 – Compromise Software Dependencies and Development Tools**

---

### Supporting Techniques

- **T1195.002 – Supply Chain Compromise**
- **T1528 – Steal Application Access Token**
- **T1552.001 – Credentials in Files**
- **T1071.001 – Web Protocols (C2/Exfiltration)**
- **T1567.001 – Exfiltration to Code Repository**

---

## 5. MITRE Data Components (DC IDs)

| Data Component | DC ID | Detection Value |
|---|---:|---|
| Process Creation | DC0032 | Detect malicious processes in CI runners |
| Command Execution | DC0064 | Detect injected shell/script execution |
| File Access | DC0055 | Detect credential file access |
| File Creation | DC0039 | Detect dropped payloads |
| Network Connection | DC0082 | Detect outbound connections |
| Network Flow | DC0078 | Identify exfil patterns |
| Network Content | DC0085 | Inspect traffic (if possible) |
| Active DNS | DC0103 | Detect malicious domains |
| Cloud Service Modification | DC0069 | Detect follow-on abuse |
| Cloud Storage Modification | DC0023 | Detect data staging/exfil |

---

## 6. Defensive Analysis

This attack proves:

> **CI/CD pipelines are high-value targets equivalent to production systems.**

### Key Defensive Takeaways

#### 1. Pin Actions to Immutable SHAs
```yaml
# UNSAFE
uses: aquasecurity/trivy-action@v0.35.0

# SAFE
uses: aquasecurity/trivy-action@<full-commit-sha>
