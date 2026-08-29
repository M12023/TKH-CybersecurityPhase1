# Phase 1 TEPP — Full-Spectrum Penetration Test

## 1. Project / Problem Statement
This project simulates a complete penetration testing engagement across three isolated lab network segments (reconnaissance, breach, and exploitation), each seeded with realistic, compounding misconfigurations. The goal was to work through a full attacker kill chain — from initial network reconnaissance through credential compromise to application-layer exploitation and data exfiltration — and then remediate every vulnerability discovered along the way, treating the exercise as both an offensive and defensive security assessment.

## 2. Approach & Architecture
The engagement was broken into four phases:

- **Phase 0 — Reconnaissance:** Full-port Nmap scans (`nmap -sV -sC -p-`) across three subnets identified live hosts and running services, including inconsistent nginx patch levels, an unauthenticated Redis instance, an insecurely configured FTP server, and a container running with excessive privileges. Cross-referencing SSH host key fingerprints across subnets revealed a critical network segmentation failure — a single host was reachable from all three "isolated" segments.
- **Phase 1 — Rapid Triage:** Each misconfigured service identified in recon was confirmed and remediated directly: enforcing a Redis password, disabling insecure vsftpd write/chroot settings, and locking down an overly permissive `/tmp` directory in a root-owned container.
- **Phase 2 — The Breach:** With `PermitRootLogin` and `PasswordAuthentication` both enabled on a target host's SSH service and no lockout policy in place, a credential attack successfully authenticated as root. The response combined an immediate `iptables` block with root-cause SSH hardening recommendations (key-only auth, `fail2ban`, IP allow-listing, SIEM alerting).
- **Phase 3 — Full Spectrum Exploitation:** A vulnerable Flask web application was found to concatenate user input directly into SQL queries. This was exploited two ways: an authentication bypass (`admin'--`) and a UNION-based SQL injection to exfiltrate salary data directly from the application's database.

## 3. Tools & Technologies
- **Reconnaissance:** Nmap (full-port version/service scanning)
- **Services Assessed/Hardened:** Redis, vsftpd (FTP), Docker containers, OpenSSH
- **Exploitation:** SQL Injection (authentication bypass, UNION-based data extraction), `curl`
- **Defensive Response:** `iptables`, SSH hardening, `redis-cli`, Linux file permissions

## 4. Results & Outcomes
- Identified and remediated four distinct misconfigurations across the triage network: unauthenticated Redis, an insecure FTP configuration, an over-privileged container, and inconsistent patch management.
- Discovered a critical network segmentation failure via SSH host key fingerprint correlation across supposedly isolated subnets.
- Successfully executed and documented a full attack chain — credential compromise via SSH, followed by authentication bypass and data exfiltration via SQL injection — entirely within the lab environment.
- Extracted evidence at every stage (before/after configuration states, process IDs, access logs, exfiltrated data) to support a complete forensic timeline.
- Concluded that the single most effective control across the entire kill chain would have been application-layer input sanitization, since it would have neutralized the final exploitation step regardless of upstream network or authentication weaknesses.

## 5. Project Evidence
Full technical write-up, including all commands, before/after remediation states, and forensic evidence for every phase: [Phase 1 Final Reckoning — TEPP Post-Mortem](https://github.com/M12023/TKH-CybersecurityPhase1/blob/main/week-12/tepp_postmortem.md)

## 6. Individual Contribution
This was solo work. I independently performed all reconnaissance, vulnerability triage and remediation, the credential attack, and the SQL injection exploitation and analysis documented in the linked postmortem, and can speak to every technical decision and finding in an interview.
