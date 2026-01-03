# Weekly Cyber Intelligence & Defensive Design Review

**Week of:** 2017-06-27 (Retrospective Example)  
**Author:** Andre Powell  
**Focus Environment:** Enterprise | Mission / Isolated

---

## 1. High-Impact Incident Selection

**Incident Name:** NotPetya (2017)  
**Category:** Worm | Supply Chain | Destructive Malware  
**Why This Matters:**  
NotPetya demonstrated that a trusted software update combined with wormable lateral movement can cause catastrophic, irreversible damage faster than detection and response can react, fundamentally changing how organizations think about prevention, segmentation, and update trust.

---

## 2. Incident Decomposition

**Summary (max 3 sentences):**
- **Initial Access:** Malicious update delivered through a compromised Ukrainian accounting software supply chain (M.E.Doc).
- **Propagation / Lateral Movement:** Automated worm-like spread using SMB exploitation (EternalBlue) and credential-based lateral movement.
- **Impact:** Rapid, irreversible destruction of Windows systems by corrupting the Master File Table, resulting in widespread operational outages.

---

## 3. MITRE ATT&CK Technique Mapping (Spine Only)

| ATT&CK Tactic        | Technique ID | Technique Name                          | Purpose |
|---------------------|--------------|------------------------------------------|---------|
| Initial Access      | T1195        | Supply Chain Compromise                  | Malware delivery via trusted update |
| Lateral Movement    | T1210        | Exploitation of Remote Services          | Wormable SMB propagation |
| Credential Access   | T1003        | OS Credential Dumping                    | Accelerated lateral spread |
| Impact              | T1485        | Data Destruction                         | Irreversible system damage |

---

## 4. Control Analysis (Prevent / Detect / Contain / Recover)

| Technique | Prevent | Detect | Contain | Recover |
|----------|---------|--------|---------|---------|
| T1195 Supply Chain Compromise | Update provenance validation; staged updates | Limited | N/A | Rebuild systems |
| T1210 Exploitation of Remote Services | Patch SMB vulnerabilities; disable SMBv1 | IDS/EDR alerts | Network segmentation | Reimage |
| T1003 Credential Dumping | Credential isolation; LSASS protection | EDR detections | Limit admin reuse | Credential reset |
| T1485 Data Destruction | N/A | Too late | N/A | Restore from backups (if available) |

---

## 5. Derived Design Requirements

### DR-1
**The system SHALL:**  
Validate and stage all software updates through an internal trusted pipeline before introduction into enterprise or mission environments.

**Rationale:**  
NotPetya entered environments through a trusted update mechanism, bypassing perimeter and detection controls entirely.

**Verification Method:**  
Audit update pipeline, verify checksum/signature validation, and conduct update integrity tabletop exercise.

---

### DR-2
**The system SHALL:**  
Prevent unrestricted lateral movement by enforcing internal network segmentation and disabling legacy protocols such as SMBv1.

**Rationale:**  
Wormable lateral movement enabled NotPetya to outpace human detection and response.

**Verification Method:**  
Network flow validation and simulated lateral movement testing.

---

## 6. Environment Gap Assessment

| Control Area | Status | Notes |
|-------------|--------|-------|
| Patch discipline | Risk | Legacy systems increased exposure |
| Segmentation | Risk | Flat internal networks enabled worm spread |
| Update provenance | Risk | Trusted update path abused |
| Credential hygiene | Risk | Shared credentials accelerated spread |
| Detection speed | Risk | Detection occurred after damage |

---

## 7. Leadership Takeaway (One Sentence)

> NotPetya showed that trusted update paths and internal lateral movement present greater systemic risk than external attackers, making prevention and architectural controls more critical than detection alone.

---

## 8. Carry-Forward Items

- [ ] Review software update trust and staging processes
- [ ] Identify and eliminate wormable internal protocols
- [ ] Validate internal segmentation boundaries
- [ ] Test recovery assumptions against destructive malware scenarios
