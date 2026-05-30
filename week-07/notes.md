# Week 07 — Session Notes
**Operator:** Maurice Ratiff III
**Topic:** The Perimeter — Reconnaissance, Scanning, and
Vulnerability Analysis
**Date:** May 28th 2026

---

## Summary

Week 7 marked the opening of Sprint 2 and a fundamental shift in
perspective — from builder and defender to scout and analyst. Where
the first six weeks developed the competencies required to construct
and harden enterprise infrastructure, this week introduced Phase 1
of the Cyber Attack Lifecycle: Reconnaissance. Across three sessions,
coursework covered passive intelligence gathering through open-source
tools, active network scanning and service enumeration using Nmap,
and vulnerability triage using the Risk equals Likelihood times Impact
framework. The week established the foundational principle of
offense-informed defense: that understanding how an attacker maps a
target is prerequisite to building defenses that can withstand that
mapping.

---

## Session 19: The Invisible Scout
(OSINT & Passive Reconnaissance)

### Summary

Session 19 introduced Open Source Intelligence (OSINT) as the first
phase of the reconnaissance process, establishing that a significant
volume of actionable intelligence about any organization is available
through public sources without sending a single packet to the target's
infrastructure. The session covered subdomain enumeration, technology
stack fingerprinting, credential leak investigation, and exposure
discovery through Shodan, producing a structured threat profile as
the session artifact.

### Key Concepts

**Open Source Intelligence (OSINT):** OSINT refers to the collection
and analysis of information from publicly available sources, including
DNS records, web archives, certificate transparency logs, data breach
databases, and internet-connected device search engines. Because
passive reconnaissance generates no traffic on the target's network,
it cannot be detected by the target's intrusion detection systems
or firewall logs, making it the preferred first phase of any
professional security assessment (Peltier, 2016).

**Subdomain Enumeration:** Subdomains represent additional attack
surface beyond the primary domain — development environments, staging
servers, administrative panels, and legacy applications are frequently
exposed through subdomains that receive less security attention than
the primary domain. Tools such as Sublist3r query certificate
transparency logs, DNS brute-force databases, and search engine
indices to enumerate subdomains without querying the target's DNS
servers directly.

**Technology Stack Fingerprinting:** Identifying the web frameworks,
content management systems, server software, and third-party
libraries in use by a target enables an attacker or security assessor
to narrow the field of applicable vulnerabilities significantly.
Browser extensions such as Wappalyzer and web services such as
BuiltWith analyze HTTP response headers, HTML source patterns, and
JavaScript library signatures to identify the technology stack
without active probing (Engebretson, 2013).

**Credential Leak Investigation:** Data breaches that expose user
credentials are catalogued by services such as HaveIBeenPwned, which
maintains a searchable database of email addresses and passwords
from known breach events. An organization whose employee email
addresses appear in breach data is at elevated risk of credential
stuffing attacks, in which leaked passwords are tested against the
organization's authentication systems.

**Shodan:** Shodan is a search engine that indexes internet-connected
devices by their network-facing banners and service responses rather
than their web content. Security professionals use Shodan to identify
exposed services, unpatched systems, and misconfigured devices
associated with a target organization by filtering results by IP
range, organization name, geography, and known vulnerability
identifiers (Matherly, 2015).

### Tools Used

| Tool | Purpose |
|------|---------|
| Sublist3r | Enumerate subdomains via public data sources |
| Wappalyzer / BuiltWith | Fingerprint target technology stack |
| HaveIBeenPwned | Identify credential leaks associated with target |
| Shodan | Discover internet-exposed services and devices |
| whois / dig | Query domain registration and DNS records |

### Lab Observations

The session build produced a structured threat profile for the
fictional target CloudNano, documented in
`ThreatProfile_CloudNano.md`. Sublist3r enumerated multiple
subdomains including what appeared to be a staging environment and
an administrative portal not linked from the primary site. Wappalyzer
identified the target's content management system and JavaScript
framework versions, enabling cross-reference against known CVEs for
those versions. A Shodan search filtered by the target's IP range
revealed an exposed service running an outdated software version
flagged by a known vulnerability identifier. The completed threat
profile documented all findings in a structured format suitable for
handoff to the active scanning phase.

### Connection to Defensive Practice

OSINT awareness is as valuable to defenders as it is to attackers.
An organization's security team should conduct regular passive
reconnaissance against their own infrastructure to understand what
an attacker can discover before launching an active engagement.
Findings such as exposed subdomains, outdated technology stack
components, and employee credentials in breach databases represent
actionable remediation items that can be addressed before an
attacker exploits them. Monitoring certificate transparency logs
and Shodan for unauthorized exposures is a standard component of
mature external attack surface management programs.

---

## Session 20: Mapping the Shadows
(Active Scanning & Enumeration)

### Summary

Session 20 transitioned from passive intelligence gathering to active
network scanning using Nmap, the industry-standard tool for host
discovery, port enumeration, and service version identification.
The session introduced the distinction between passive and active
reconnaissance, established the legal and ethical boundaries of
active scanning, and produced a professional network enumeration
map of a sandboxed Docker target network.

### Key Concepts

**Active Reconnaissance:** Active reconnaissance involves sending
packets directly to the target's systems to elicit responses that
reveal information about the network topology, live hosts, and
running services. Unlike passive reconnaissance, active scanning
generates traffic on the target's network and may trigger alerts
in intrusion detection systems. Active scanning against systems
without explicit written authorization is illegal under the Computer
Fraud and Abuse Act in the United States and equivalent legislation
in other jurisdictions (US Department of Justice, 2020).

**Nmap:** Nmap (Network Mapper) is an open-source tool for network
discovery and security auditing. It uses raw IP packets to determine
which hosts are available on a network, which ports are open on
those hosts, which services are running on those ports, and which
operating systems the hosts are running (Lyon, 2009). Nmap is the
most widely used network scanning tool in both offensive security
assessments and defensive network auditing.

**Ping Sweep:** A ping sweep sends ICMP echo requests to every
address in a specified subnet to identify which hosts are live.
The Nmap flag `-sn` performs host discovery without port scanning,
producing a list of responsive hosts that serves as the input for
subsequent, more detailed scans.

**Service Version Enumeration:** The Nmap flag `-sV` instructs the
scanner to probe open ports and identify the specific software and
version running on each service. Version information is critical
for vulnerability assessment because many vulnerabilities are
version-specific — knowing that a host is running Apache 2.4.49
rather than simply "an HTTP server" enables direct lookup of
applicable CVEs.

**Network Enumeration Map:** A network enumeration map is a
structured document recording all discovered hosts, their open
ports, and their identified service versions. This artifact serves
as the foundation for the vulnerability analysis phase and as
evidence of scope compliance in a formal penetration test engagement.

### Tools Used

| Tool | Purpose |
|------|---------|
| `nmap -sn` | Ping sweep to identify live hosts on a subnet |
| `nmap -sV` | Service version enumeration on discovered hosts |
| `nmap -sT` | TCP connect scan for full port enumeration |
| `nmap -p-` | Scan all 65,535 ports rather than top 1,000 |
| `nmap -oN` | Save scan output to a file for documentation |

### Lab Observations

The session build deployed a sandboxed Docker network on the
`172.99.0.0/24` subnet and executed a sequential scanning workflow.
The initial ping sweep with `nmap -sn 172.99.0.0/24` identified
the live hosts within the subnet. Version scans with `nmap -sV`
against each discovered host revealed the specific services and
software versions running on open ports, including web server
versions and database service identifiers. All findings were
documented in `nmap_scan_results.txt` as a structured enumeration
map organized by host IP, open port, protocol, and identified
service version.

### Connection to Defensive Practice

Nmap is not exclusively an offensive tool — network defenders use
it routinely for asset discovery, configuration auditing, and
change detection. Running periodic Nmap scans against internal
subnets and comparing results against a known-good baseline reveals
unauthorized devices, newly exposed services, and configuration
changes that may indicate compromise or misconfiguration. The same
scan syntax used in this session to map a target network is used
by security operations teams to audit their own infrastructure for
unexpected exposures.

---

## Session 21: The Prioritization Matrix
(Vulnerability Scanning)

### Summary

Session 21 introduced automated vulnerability scanning using Nikto
and the application of the Risk equals Likelihood times Impact
framework to triage a realistic vulnerability report. The session
emphasized that a vulnerability's CVSS score is an input to risk
assessment rather than a complete risk determination, and that
effective remediation planning requires contextual judgment about
exploitability, asset criticality, and compensating controls.

### Key Concepts

**Nikto:** Nikto is an open-source web server scanner that tests
targets for thousands of known vulnerabilities, misconfigurations,
and information disclosure issues. It checks for outdated server
software, dangerous HTTP methods, default credentials, exposed
administrative interfaces, and common web application weaknesses.
Nikto output provides a rapid initial assessment of a web server's
security posture and identifies items requiring deeper investigation
(Sullo & Lodge, 2012).

**CVE and CVSS:** A Common Vulnerability and Exposure (CVE) entry
is a standardized identifier for a specific, publicly known
vulnerability. Each CVE is assigned a Common Vulnerability Scoring
System (CVSS) score between 0 and 10 that reflects the severity
of the vulnerability based on factors including attack vector,
attack complexity, required privileges, and potential impact.
However, CVSS scores reflect the theoretical severity of a
vulnerability in isolation and do not account for the specific
context of the affected organization, making them an incomplete
basis for remediation prioritization (NIST, 2023).

**Risk = Likelihood × Impact:** The Risk equals Likelihood times
Impact formula provides a framework for contextualizing
vulnerability severity. Likelihood reflects the probability that
a specific vulnerability will be exploited in the target
environment, accounting for factors such as public exploit
availability, attacker motivation, and existing compensating
controls. Impact reflects the consequence of successful
exploitation for the specific asset, accounting for the asset's
criticality, the data it processes, and the regulatory obligations
associated with a breach. Applying this formula to a vulnerability
list produces a risk-ranked remediation order that reflects
operational reality rather than theoretical severity scores.

**Remediation Prioritization:** Effective remediation planning
does not simply address the highest CVSS scores first. A critical-
severity vulnerability in an isolated internal system with no
public exposure may represent lower actual risk than a medium-
severity vulnerability in an internet-facing authentication portal
with a publicly available exploit. Professional remediation plans
document the justification for each prioritization decision,
enabling stakeholders to understand and validate the risk
assessment logic (NIST, 2018).

### Tools Used

| Tool | Purpose |
|------|---------|
| Nikto | Automated web vulnerability scanning |
| `docker run` | Deploy vulnerable target container for scanning |
| CVSS Calculator | Score individual vulnerabilities for comparison |
| NVD / CVE Database | Research vulnerability details and patch status |

### Lab Observations

The session deployed a deliberately vulnerable Docker container as
the scan target and ran Nikto against its web service, producing
a raw finding list containing multiple vulnerabilities of varying
severity. The triage exercise applied the Risk equals Likelihood
times Impact formula to a 20-item CloudNano remediation scenario,
reducing the full finding list to the five items representing the
greatest actual risk to the organization. Each selection was
justified in writing in `remediation_plan.md`, documenting the
likelihood and impact assessment for each prioritized finding and
explaining why higher-scored items were ranked below lower-scored
ones in cases where contextual factors reduced their real-world
exploitability.

### Connection to Defensive Practice

Vulnerability triage is one of the most consequential skills in
security operations because remediation resources are always
finite. An organization cannot patch every vulnerability
simultaneously, and a triage methodology that relies on CVSS
scores alone will consistently misallocate resources toward
theoretical risks while leaving high-probability, high-impact
exposures unaddressed. The Risk equals Likelihood times Impact
framework is the foundation of mature vulnerability management
programs and is embedded in standards including the NIST
Cybersecurity Framework and CIS Controls (NIST, 2018). Developing
the judgment to apply this framework accurately is a core
competency for security analysts, engineers, and program managers
operating in any sector.

---

## References

Engebretson, P. (2013). *The basics of hacking and penetration
    testing: Ethical hacking and penetration testing made easy*
    (2nd ed.). Syngress.

Lyon, G. (2009). *Nmap network scanning: The official Nmap project
    guide to network discovery and security scanning*. Insecure.com.
    https://nmap.org/book/

Matherly, J. (2015). *Complete guide to Shodan*. Leanpub.
    https://leanpub.com/shodan

NIST. (2018). *Framework for improving critical infrastructure
    cybersecurity (version 1.1)*.
    https://nvlpubs.nist.gov/nistpubs/CSWP/NIST.CSWP.04162018.pdf

NIST. (2023). *National vulnerability database*.
    https://nvd.nist.gov/

Peltier, T. R. (2016). *Information security policies, procedures,
    and standards: Guidelines for effective information security
    management*. Auerbach Publications.

Sullo, C., & Lodge, D. (2012). *Nikto web server scanner*.
    https://cirt.net/Nikto2

US Department of Justice. (2020). *Prosecuting computer crimes*.
    https://www.justice.gov/sites/default/files/criminal-ccips/
    legacy/2015/01/14/ccmanual.pdf
