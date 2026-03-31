# SEC511 Weekly Cyber Intelligence Journal  
## Week of April 6 – Axios NPM Supply Chain Compromise (March 31, 2026)

---

## Executive Summary

On March 31, 2026, attackers compromised the widely used JavaScript library Axios on the npm registry, injecting malicious code into versions 1.14.1 and 0.30.4.

Rather than modifying Axios source code, the attackers introduced a malicious dependency (`plain-crypto-js`) that executed automatically during installation via npm lifecycle scripts. This payload harvested credentials and deployed a cross-platform Remote Access Trojan (RAT).

With over 80–100 million weekly downloads, the impact spans developer workstations, CI/CD pipelines, and production environments globally.

> This incident demonstrates a critical reality: **modern attacks target the software supply chain—not the application itself.**

---

## What Happened

- Attackers compromised the npm account of an Axios maintainer
- Published malicious versions:
  - `axios@1.14.1`
  - `axios@0.30.4`
- Injected dependency:
  - `plain-crypto-js@4.2.1`
- Attack triggered via:
  - **npm postinstall script execution**
- Exposure window:
  - ~2–3 hours before removal

Sources:  
- :contentReference[oaicite:0]{index=0}  
- :contentReference[oaicite:1]{index=1}  

---

## Key Technical Insight (What Makes This Attack Dangerous)

> The attacker never modified Axios itself — the entire attack lived in a transitive dependency executed automatically during installation.

- Malicious behavior triggered **without user interaction**
- Hidden in dependency chain → bypasses code review
- Executed via **npm lifecycle (postinstall)**
- Self-deletes and replaces metadata to evade detection

This is a **next-level supply chain attack pattern**

---

## Attack Chain (Detailed)

1. Compromise npm maintainer credentials (likely token theft)  
2. Pre-stage malicious dependency (`plain-crypto-js`)  
3. Publish poisoned Axios versions across multiple branches  
4. Developer/CI system runs `npm install`  
5. Postinstall script executes automatically  
6. Dropper determines OS and deploys platform-specific payload  
7. Credentials exfiltrated immediately  
8. RAT installed and begins beaconing every 60 seconds  
9. Malware deletes itself and cleans artifacts  

---

## Payload Behavior (Cross-Platform)

### macOS
- Drops binary: `/Library/Caches/com.apple.act.mond`
- Executes via AppleScript + zsh
- C++ RAT with beaconing + command execution

### Windows
- Copies PowerShell → `%PROGRAMDATA%\wt.exe`
- Executes via VBScript + PowerShell
- Persistence:
  - `%PROGRAMDATA%\system.bat`
  - Registry Run Key

### Linux
- Drops Python RAT: `/tmp/ld.py`
- Executes via `nohup`
- No built-in persistence (likely second-stage deployment)

Source: :contentReference[oaicite:2]{index=2}  

---

## Command & Control

- Domain: `sfrclak.com`
- IP: `142.11.206.73`
- Port: `8000`
- Callback paths:
  - `/product0` (macOS)
  - `/product1` (Windows)
  - `/product2` (Linux)

- Beacon interval: ~60 seconds
- Supports:
  - Command execution
  - File system enumeration
  - Payload delivery

---

## MITRE ATT&CK Mapping

| Tactic | Technique | ID |
|------|----------|----|
| Initial Access | Supply Chain Compromise | T1195 |
| Execution | Command and Scripting Interpreter | T1059 |
| Execution | User Execution | T1204 |
| Persistence | Boot or Logon Autostart Execution | T1547 |
| Persistence | Logon Initialization Scripts | T1037 |
| Credential Access | OS Credential Dumping | T1003 |
| Credential Access | Unsecured Credentials | T1552 |
| Defense Evasion | Obfuscated/Compressed Files | T1027 |
| Defense Evasion | Indicator Removal on Host | T1070 |
| Defense Evasion | Masquerading | T1036 |
| Command & Control | Application Layer Protocol | T1071 |
| Command & Control | Web Protocols | T1071.001 |
| Discovery | Cloud Infrastructure Discovery | T1580 |

---

## MITRE Data Components (DCs)

| Data Component | DC ID |
|---------------|------|
| Process Creation | DC0001 |
| File Creation | DC0002 |
| File Modification | DC0003 |
| Network Connection Creation | DC0005 |
| Command Execution | DC0017 |
| Logon Session Creation | DC0009 |
| Script Execution | DC0018 |
| Application Log Content | DC0015 |
| Cloud API Activity | DC0065 |

---

## Detection Opportunities

### Host-Based Detection

- Execution of unexpected scripts during package install
- Suspicious files:
  - `/tmp/ld.py`
  - `%TEMP%\6202033.ps1`
  - `/Library/Caches/com.apple.act.mond`
- Modified shell profiles:
  - `.bashrc`, `.zshrc`
- Creation of:
  - `%PROGRAMDATA%\wt.exe`
  - Registry Run keys

---

### Network Detection

- Outbound traffic:
  - `142.11.206.73:8000`
  - `sfrclak.com`
- Suspicious POST requests:
  - `packages.npm.org/product0/1/2`
- Beaconing every ~60 seconds

---

### Cloud Detection

- Unusual API usage after build events:
  - AWS IAM activity (CreateUser, AssumeRole)
  - Secrets access
  - S3 enumeration
- CI/CD runners making outbound connections

---

### Supply Chain Detection (CRITICAL)

- Unexpected dependency:
  - `plain-crypto-js`
- New dependency not referenced in source code
- Postinstall scripts in dependencies
- Changes in package-lock.json / yarn.lock

---

## Defensive Design Insights

### Prevent

- Pin dependencies (lockfiles enforced)
- Use:
  - `npm ci` instead of `npm install`
- Disable lifecycle scripts in CI/CD:
  ```bash
  npm install --ignore-scripts
