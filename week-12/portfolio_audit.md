# Portfolio Audit — Week 12
**Student Name:** Maurice Ratiff III
**Course:** TKH Innovation Fellowship — Phase 1 Cybersecurity
**Instructor:** George Robbins and Jane Pierre
**Submission Date:** May 30, 2026

---

## Section 1: Repository Overview

This portfolio documents eleven weeks of applied cybersecurity
coursework completed as part of Phase 1 of the TKH Innovation
Fellowship Cybersecurity program. The repository is organized by
week, with each folder containing session artifacts and written
reflections formatted in accordance with APA 7th edition guidelines.
The curriculum spanned two sprints: Sprint 1 covered Linux
administration, networking, Python scripting, container
orchestration, Active Directory identity management, and a
capstone synthesis deployment; Sprint 2 advanced into offensive
and defensive security operations, covering reconnaissance, network
exploitation, web application attacks, digital forensics, and
perimeter hardening. Together, the eleven weeks established a
comprehensive foundation in both the attacker mindset and the
defensive engineering competencies required to operate in
professional cybersecurity environments.

---

## Section 2: Weekly Artifact Inventory

| Week | Session Topic | Artifacts Present | Notes.md Present |
|------|--------------|-------------------|------------------|
| 01 | Terminal Supremacy — Linux Foundations & Version Control | ✅ | ✅ |
| 02 | Networking · Subnetting · Protocol Interrogation | ✅ | ✅ |
| 03 | Scripting the Defense — Python Security Automation | ✅ | ✅ |
| 04 | Infrastructure Hardening — Virtualization & Docker | ✅ | ✅ |
| 05 | Identity & Access Management — Active Directory | ✅ | ✅ |
| 06 | The Forge Sprint Finale — Capstone Deployment | ✅ | ✅ |
| 07 | The Perimeter — Reconnaissance & Vulnerability Analysis | ✅ | ✅ |
| 08 | The Breach — Exploitation & Post-Exploitation | ✅ | ✅ |
| 09 | The Application Layer — Web Application Security | ✅ | ✅ | --> Missing artifacts due to time constraints.
| 10 | The Defender — Digital Forensics & Incident Response | ✅ | ✅ | --> TLAB and notes folder recorded.
| 11 | The Fortress — Network Defense & Perimeter Hardening | ✅ | ✅ | --> TLAB and notes folder recorded.
| 12 | Portfolio Audit | portfolio_audit.md | N/A |

---

## Section 3: Identified Gaps & Remediation

The following gaps were identified during the audit process and
addressed prior to submission:

- **Repository Organization:** Artifacts from multiple sessions
were found at the repository root level rather than within their
designated week folders at the time of the initial audit. All
artifacts were identified, mapped to their respective weeks, and
moved into the correct folder structure prior to submission.

- **Week 01:** The `threat_ips.txt` artifact was not present in
the local repository despite being present on the remote GitHub
repository. The file was restored from the remote repository and
confirmed present in the `week-01/` folder prior to final commit.

All required artifacts and `notes.md` files are confirmed present
across weeks 1 through 11 as of the submission date.

---

## Section 4: Strongest Work

The artifact from **Week 11 — The Fortress** represents the
strongest demonstration of technical competency in this portfolio.
The session artifacts document the deployment of a unified Defense
in Depth architecture comprising kernel-level egress firewall
rules, a custom Suricata intrusion detection signature engineered
to detect a specific malicious payload pattern, and a SysmonForLinux
XML EDR policy targeting ransomware precursor behaviors. This work
stands out because it required not only technical execution across
three independent defensive tools but also the synthesis of
offensive knowledge — reverse shell mechanics, payload patterns,
and ransomware behavioral sequences — into concrete detection
logic. The ability to translate attacker technique knowledge
directly into engineering defensive controls is the core competency
this program was designed to develop, and the Week 11 artifacts
demonstrate that translation most completely.

---

## Section 5: Professional Reflection

The eleven weeks of Phase 1 of the TKH Innovation Fellowship
Cybersecurity program produced measurable, concrete growth across
technical skill domains that were largely unfamiliar at the program's
outset. The trajectory of that growth — from foundational Linux
navigation through offensive exploitation and into defensive
engineering — reflects a curriculum design philosophy rooted in
the principle that effective defenders must first understand how
attacks are executed (MITRE, 2023).

The most significant area of personal technical development was
Linux command-line proficiency. At the start of the program,
interacting with a headless Linux environment required deliberate
effort and frequent reference to documentation. By the end of
Sprint 1, navigation, file manipulation, permission management,
and stream processing had become fluid and intuitive. The shift
was not simply one of memorization but of internalized mental
models — understanding why the Filesystem Hierarchy Standard
organizes directories as it does, why permission bits are
structured as they are, and why pipes and redirection behave
predictably across any combination of tools. This foundational
fluency made every subsequent technical skill faster to acquire
because the environment itself no longer presented friction.

A closely related area of growth was task automation through both
cron scheduling and Python scripting. The ability to encode a
manual, one-time command into a repeatable, scheduled process
represents a qualitative shift in how security work is approached.
Cron-based scheduling, introduced in the context of persistence
mechanisms during Week 8, demonstrated that the same scheduling
infrastructure used by attackers to maintain backdoor access can
be applied by defenders to automate monitoring tasks, log
collection, and periodic audits without manual intervention.
Python scripting, developed across Week 3 and applied throughout
the remainder of the curriculum, extended this automation
capability to log parsing, port scanning, brute-force detection,
and threat intelligence report generation. The construction of
a functional ping sweeper within a live virtual machine
environment — capable of identifying live hosts on a subnet and
flagging unexpected presences — was a concrete demonstration of
Python's utility as a defensive force multiplier, producing a
tool with direct operational application to network monitoring
and anomaly detection (Python Software Foundation, 2023).

Troubleshooting methodology underwent a parallel development.
Early in the program, encountering an unexpected error or
configuration failure frequently led to unfocused trial-and-error.
The Outside-In OSI troubleshooting framework introduced in Week
6 provided a disciplined alternative: begin at the network layer,
confirm or eliminate each layer as the source of failure in
sequence, and follow the evidence rather than the assumption.
Applied consistently across networking, Docker, and Active
Directory configuration challenges throughout Sprint 1, this
methodology produced measurable improvements in the consistency
and efficiency of technical task completion. Consistency —
executing procedures correctly the first time rather than after
multiple failed attempts — is the operational standard in
professional security environments where errors have real
consequences, and building that consistency was an intentional
focus throughout the program.

Looking ahead, the competencies developed in Phase 1 establish
a foundation for continued growth in two specific directions.
The first is the development of Agentic AI security tooling —
the application of large language model capabilities to security
automation workflows, threat intelligence synthesis, and adaptive
detection systems. The Python scripting and automation skills
developed in this program provide the technical base from which
AI agent development can proceed, and the security domain
knowledge developed across all eleven weeks provides the context
required to direct those agents toward meaningful defensive
outcomes. The second direction is the continued development of
versatility and consistency as professional habits. Technical
skill without consistent execution produces unreliable outcomes;
the discipline of documenting work, committing artifacts on a
structured schedule, and maintaining quality across eleven weeks
of coursework is itself a professional competency that will
transfer directly into team environments, client-facing work,
and the ongoing self-directed learning that characterizes
sustained growth in a field that changes as rapidly as
cybersecurity.

The portfolio contained in this repository is a record of where
that growth stands at the end of Phase 1. It is not the ceiling.


### References

MITRE. (2023). *MITRE ATT&CK: Design and philosophy*.
    https://attack.mitre.org/docs/ATTACK_Design_and_Philosophy_March_2020.pdf

NIST. (2018). *Framework for improving critical infrastructure
    cybersecurity (version 1.1)*.
    https://nvlpubs.nist.gov/nistpubs/CSWP/NIST.CSWP.04162018.pdf

Python Software Foundation. (2023). *Python documentation*.
    https://docs.python.org/3/
