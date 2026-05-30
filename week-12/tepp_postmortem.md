# Phase 1 Final Reckoning — TEPP Post-Mortem
**Operator:** [Your Full Name]
**Date:** May 28, 2026
**Repository:** [Your GitHub repo URL]
**TKH Innovation Fellowship 2026 | Phase 1 | Cybersecurity**

---

## Phase 0: Reconnaissance

### Triage Network — 172.100.0.0/24
[3–5 sentences in APA style. What hosts did you find? What ports and
services were exposed? What misconfigurations did you identify?]

### Breach Network — 172.80.0.0/24
[3–5 sentences in APA style. What hosts did you find? What ports and
services were exposed? What did you observe that informed your Phase 2
approach?]

### Exploitation Network — 172.60.0.0/24
[3–5 sentences in APA style. What hosts did you find? What ports and
services were exposed? What vulnerability did you identify before
executing your exploit?]

---

## Phase 1: Rapid Triage

### Server 1 — 172.100.0.11
**Vulnerability Identified:**
[What was exposed and how did you confirm it?]

**Remediation Commands:**
[Exact commands used to enter the container and apply the fix]

**Before State:**
[What did the service or permission look like before your fix?]

**After State:**
[What did it look like after?]

**Analysis:**
[2–3 sentences in APA style — why is this vulnerability dangerous
in a real enterprise environment?]

### Server 2 — 172.100.0.12
**Vulnerability Identified:**
[What unauthorized service was running and how did you confirm it?]

**Remediation Commands:**
[Exact commands used to enter the container and terminate the process]

**Before State:**
[What was running before your remediation?]

**After State:**
[What was the state after termination?]

**Analysis:**
[2–3 sentences in APA style — why is this vulnerability dangerous
in a real enterprise environment?]

### Server 3 — 172.100.0.13
**Vulnerability Identified:**
[What directory had dangerous permissions and what were they exactly?]

**Remediation Commands:**
[Exact commands used to enter the container and apply chmod]

**Before State:**
[What were the permissions before your fix? Be specific.]

**After State:**
[What were the permissions after?]

**Analysis:**
[2–3 sentences in APA style — why is this vulnerability dangerous
in a real enterprise environment?]

---

## Phase 2: The Breach

**Cracked Credentials:**
- Username: [username]
- Password: [password]

**Forensic Evidence:**
- Exact Timestamp of Successful Login: [timestamp from auth logs]
- Attacker IP Address: [IP recorded in the logs]

**Engineered iptables Rule:**
[Complete iptables command — chain, action, and target IP]

**SOC Analysis:**
[2–3 sentences in APA style — why is a single iptables block rule
insufficient as a standalone defensive measure? What additional
controls would a real SOC deploy alongside it?]

---

## Phase 3: Full Spectrum

**Listener Configuration:**
[What tool, what port, what command did you use to set up your listener?]

**Reverse Shell Payload:**
[The exact curl command you crafted to trigger the exploit]

**Command Injection Explanation:**
[2–3 sentences in APA style — how does command injection work and
why is this application susceptible to it?]

**Forensic Evidence:**
- Process ID (PID): [PID from access.log]
- User-Agent: [User-Agent string from access.log]

**Lockdown Command:**
[Exact iptables command applied inside the container]

**Final Analytical Paragraph:**
[4–6 sentences in APA style responding to: You have now played both
sides of this operation. What does executing this attack teach you
about defending against it? What single defensive control, if it had
been in place before you attacked, would have stopped this breach
entirely — and why?]

---

## References
[APA format. Any tools, documentation, or resources referenced
during this operation.
Example: Hydra Project. (2024). THC-Hydra: A fast and flexible
online password cracking tool. https://github.com/vanhauser-thc/thc-hydra]
