# TKH Innovation Fellowship — Phase 1 Cybersecurity

 M12023 - Maurice Ratiff III · Class of 2026



---

## 👋 Welcome

This is my student artifact repository for Phase 1 of the TKH
Innovation Fellowship Cybersecurity program. Every week I push
the work I build in class — scripts, lab outputs, notes, and
documentation — so there is a living record of how far I have come.

**Student:** Maurice Ratiff III
**Role:** Student · TKH Innovation Fellowship 2026
**Program:** [The Knowledge House](https://theknowledgehouse.org) · Cybersecurity
**Phase:** 1 of 2 · Spring–Summer 2026
**Cohort:** Class of 2026

---

## 📁 Repository Structure

TKH-CybersecurityPhase1/
├── week-01/
│   ├── notes.md
│   ├── discovery.txt
│   ├── harden.sh
│   ├── threat_ips.txt
│   └── final_threat_report.txt
├── week-02/
│   ├── notes.md
│   ├── network_audit.txt
│   ├── subnet_blueprint.txt
│   ├── protocol_audit.txt
│   └── tlab_report.txt
├── week-03/
│   ├── notes.md
│   ├── port_check.py
│   ├── brute_detector.py
│   └── brute_report.txt
├── week-04/
│   ├── notes.md
│   ├── sandbox_report.txt
│   ├── deploy_web.sh
│   └── docker-compose.yml
├── week-05/
│   ├── notes.md
│   ├── onboard_engineers.ps1
│   └── gpo_audit.txt
├── week-06/
│   ├── notes.md
│   ├── readiness_check.log
│   ├── practical_exam_report.txt
│   └── HardenedOutpost_SAD.md
├── week-07/
│   ├── notes.md
│   ├── ThreatProfile_CloudNano.md
│   ├── nmap_scan_results.txt
│   └── remediation_plan.md
├── week-08/
│   ├── notes.md
│   ├── Exploit_Verification.png
│   ├── escalation_path.txt
│   └── pivot_success.png
├── week-09/
│   ├── notes.md
│   └── [web app artifacts]
├── week-10/
│   ├── notes.md
│   └── Incident_Response_Report.md
├── week-11/
│   ├── notes.md
│   └── Operation_Fortress_Report.md
└── week-12/
└── portfolio_audit.md

---

## 📅 Week Tracker

| Week | Dates | Theme | Status |
|------|-------|-------|--------|
| 01 | Mar 9–11 | Terminal · Permissions · Stream Editing · Git | ✅ Complete |
| 02 | Mar 16–18 | Networking · Subnetting · Protocol Interrogation | ✅ Complete |
| 03 | Mar 23–25 | Python Scripting · Port Scanner · Brute Force Detector | ✅ Complete |
| 04 | Mar 30–Apr 1 | Virtualization · Docker · Container Security · Network Segmentation | ✅ Complete |
| 05 | Apr 6–8 | Identity · Active Directory · Windows Server Core | ✅ Complete |
| 06 | Apr 13–15 | Forge Capstone · Hybrid Architecture · Secure Deployment | ✅ Complete |
| 07 | [Dates] | Reconnaissance · OSINT · Vulnerability Analysis | ✅ Complete |
| 08 | [Dates] | Exploitation · Privilege Escalation · Post-Exploitation | ✅ Complete |
| 09 | [Dates] | Web Application Security · SQLi · XSS · API Exploitation | ✅ Complete |
| 10 | [Dates] | Digital Forensics · Incident Response · SIEM | ✅ Complete |
| 11 | [Dates] | Network Defense · IDS · EDR · Perimeter Hardening | ✅ Complete |
| 12 | May 30 | Portfolio Audit | ✅ Complete |

---

## 🗂️ Week Breakdown

---

### 🖥️ Week 01 — Linux Foundations and Version Control

#### 🌱 S01 · Terminal Genesis
First session in a headless Linux environment. No GUI. Just the terminal.

Key skills: `ls` `cd` `pwd` `mkdir` `cat` `find` · FHS navigation · SSH · Git setup

#### 🔐 S02 · The Keymaster
Who can read it? Who can write it? Who can run it? Nine bits. Three letters: `rwx`.

Key skills: `chmod` `chown` `ls -la` · SUID auditing · Principle of Least Privilege

#### 🔵 S03 · Stream Editing and Automation
10,000 lines of web server logs. Three attackers buried in the noise. One pipeline.

Key skills: `grep` `awk` `sed` `sort` `uniq` · stdout redirection · pipeline chaining

#### 🎯 TLAB-01 · Operation Clean Sweep
Extracted malicious IPs from web logs, correlated with auth logs, filed threat report.

---

### 🌐 Week 02 — Networking · Subnetting · Protocol Interrogation

#### 🔌 S04 · The Wire (L1–L3)
Restore a blind terminal to the grid by rebuilding downed interfaces and routes.

Key skills: `ip link` `ip addr` `ip route` · Layer 1–3 recovery · default gateway

#### 🧮 S05 · The Subnetting Crucible
Master CIDR notation and the Magic Number to define secure network boundaries.

Key skills: `ipcalc` · binary conversion · CIDR · subnet masks · Python `bin()`

#### 🔍 S06 · Protocol Interrogation
Audit the application layer, remediate DNS deception, perform service discovery.

Key skills: `ss` `dig` `curl` · `/etc/hosts` remediation · non-standard port detection

#### 🎯 TLAB-02 · Operation Silent Ghost
Full SOC analyst incident response: downed link, subnet fix, hidden exfil port capture.

---

### 🐍 Week 03 — Scripting the Defense

#### 🛡️ S07 · The Sentry
Python foundations: data types, casting, virtual environments, port scanner.

Key skills: `venv` · `socket` library · `input()` · `int()` · TypeErrors

#### 📋 S08 · The Paper Trail
Autonomous decision logic and loops to process security data at scale.

Key skills: Python `list` · `in` operator · `for` loops · `if/else` · IP blacklisting

#### ⚙️ S09 · The Conductor
Functions, file I/O, exception handling, and automated log parsing.

Key skills: `def` · `open()` · `try/except` · `readlines()` · structured output files

#### 🎯 TLAB-03 · Operation Python Sentry
Built a complete log-parsing sentry producing a formatted Threat Intelligence Report.

---

### 🐳 Week 04 — Infrastructure Hardening

#### 👻 S10 · The Ghost in the Machine
Hypervisor architecture, virtual networking modes, forensic sandbox deployment.

Key skills: VirtualBox · Bridged vs Host-Only · Type 1/2 hypervisors · detonation testing

#### 📦 S11 · The Container Revolution
Docker fundamentals: images, containers, namespaces, disposable infrastructure.

Key skills: `docker pull` `docker run` `docker logs` `docker rm` · process isolation

#### 🎼 S12 · The Conductor and the Fleet
Multi-container orchestration with YAML and air-gapped network segmentation.

Key skills: `docker-compose` · YAML · bridge networks · `internal: true` isolation

---

### 🏢 Week 05 — Identity & Access Management

#### 🧠 S13 · The Corporate Brain
Promote Windows Server to Domain Controller, build AD hierarchy, automate users.

Key skills: Active Directory · `New-ADUser` · PowerShell · Forests · OUs · Kerberos

#### 👁️ S14 · The Invisible Hand
Enforce system-wide security policies through GPOs and master LSDOU inheritance.

Key skills: GPO · `gpupdate /force` · `gpresult` · LSDOU · policy conflict resolution

#### 🌉 S15 · Bridging the Kingdoms
Join Ubuntu to Active Directory domain, configure SSSD, build sudoers bridge.

Key skills: `realm join` · `sssd` · `visudo` · cross-platform authentication

---

### ⚙️ Week 06 — The Forge Sprint Finale

#### 🏗️ S16 · The Architect's War Room
Diagnose and repair a broken environment using the Outside-In OSI methodology.

Key skills: `ping` `nc` `ls -la` `systemctl` · Layer 3/4/7 diagnostics

#### 🏆 S17 · The Forge Final
Sprint 1 mastery exam across Linux, Networking, Python, Docker, and Active Directory.

Key skills: All Sprint 1 domains · hidden file discovery · permission lockdown

#### 🚀 S18 · The Capstone — The Hardened Outpost
Solo 3-hour deployment of a full enterprise environment for Titan Small Business.

Key skills: Full-stack integration · Security Architecture Document · SAD writing

---

### 🔭 Week 07 — The Perimeter

#### 🕵️ S19 · The Invisible Scout
Map an organization's full digital footprint without sending a single packet.

Key skills: Sublist3r · Wappalyzer · Shodan · HaveIBeenPwned · passive recon

#### 🗺️ S20 · Mapping the Shadows
Active scanning and service enumeration against a sandboxed Docker network.

Key skills: `nmap -sn` `nmap -sV` `nmap -sT` · ping sweep · enumeration mapping

#### 🎯 S21 · The Prioritization Matrix
Run Nikto, triage a 20-item report down to the 5 highest actual-risk findings.

Key skills: Nikto · CVE/CVSS · Risk = Likelihood × Impact · remediation planning

---

### 💥 Week 08 — The Breach

#### ✅ S22 · The Verification Protocol
Catch a reverse shell with Netcat, breach a legacy server via EternalBlue/MS17-010.

Key skills: `nc -lvnp` · Metasploit · `msfconsole` · Meterpreter · EternalBlue

#### 🪜 S23 · Climbing the Ladder
Escalate from restricted foothold to full SYSTEM via GTFOBins and unquoted paths.

Key skills: `sudo -l` · GTFOBins · WinPEAS · unquoted service path · privilege escalation

#### 🌐 S24 · The Deep Network
Persistent cron backdoor and network pivot through a compromised host.

Key skills: `crontab` · autoroute · SOCKS proxy · proxychains · lateral movement

---

### 🌐 Week 09 — The Application Layer

#### 💾 S25 · The Data Exfiltration
Break login portals, map database schemas, extract data via SQL Injection.

Key skills: SQLi · UNION attacks · `information_schema` · authentication bypass

#### ☠️ S26 · The Poisoned Browser
Inject malicious JavaScript, steal session cookies, craft CSRF attack links.

Key skills: Reflected XSS · Stored XSS · `document.cookie` · CSRF · DOM manipulation

#### 🔌 S27 · The Invisible Logic
Intercept live API traffic, exploit BOLA, brute-force hidden parameters.

Key skills: Burp Suite · REST API · BOLA/IDOR · Intruder · Man-in-the-Middle proxy

Note: Due to time constraints Week 09 Artifacts are not present.
---

### 🔍 Week 10 — The Defender (DFIR)

#### 🚨 S28 · The Crime Scene
Live response, volatile evidence collection, cryptographic chain of custody.

Key skills: `ps aux` `ss` `lsof` · `sha256sum` · PICERL · order of volatility

#### 🩻 S29 · The Digital Autopsy
Memory forensics, disk imaging, and deleted file recovery with The Sleuth Kit.

Key skills: `dd` · `fls` `icat` `mmls` · MFT analysis · file carving

#### 🧠 S30 · The Central Nervous System
SIEM navigation, log correlation, and full attack timeline reconstruction.

Key skills: Kibana · KQL · ELK Stack · log correlation · attack timelining

Week 10 TLAB Recorded.
---

### 🏰 Week 11 — The Fortress

#### 🏗️ S31 · The Barricade
DMZ architecture, kernel-level firewall engineering, and egress filtering.

Key skills: `iptables` · UFW · egress rules · DMZ · Docker internal networks

#### 🪤 S32 · The Tripwire
Deploy Suricata IDS and engineer custom signatures to detect malicious payloads.

Key skills: Suricata · custom rules · `fast.log` · signature engineering

#### 🔬 S33 · The Last Mile
SysmonForLinux process monitoring and XML EDR policy for ransomware detection.

Key skills: SysmonForLinux · XML policy · process creation events · EDR

Week 11 Recorded.
---

## 🚀 Getting Started

---

## 🛠️ Skills Demonstrated

- Linux command-line proficiency and headless environment navigation
- Network engineering, subnetting, and protocol analysis
- Python security automation and log parsing tooling
- Container orchestration and infrastructure hardening
- Active Directory identity and access management
- Offensive security: reconnaissance, exploitation, web attacks
- Digital forensics and incident response
- Network defense: firewall engineering, IDS, and EDR deployment
- Technical documentation in APA style
