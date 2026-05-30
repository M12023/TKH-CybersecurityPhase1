# Week 06 — Session Notes
**Operator:** Maurice Ratiff III
**Topic:** The Forge Sprint Finale — Synthesis, Diagnostics, and
Capstone Deployment
**Date:** May 29th 2026

---

## Summary

Week 6 concluded The Forge sprint by shifting from structured
instruction to applied synthesis under pressure. Rather than
introducing new tools or concepts, the week required demonstration
of mastery across all five domains covered in the preceding five
weeks: Linux administration, networking, Python scripting, container
orchestration, and Active Directory identity management. Session 16
presented a deliberately broken environment requiring OSI-layer
diagnostic methodology. Session 17 assessed Sprint 1 competency
through a timed theory quiz and an unguided practical examination.
Session 18 was a solo capstone deployment of a fully integrated,
cross-platform enterprise environment built from a blank slate within
a three-hour window. Completion of all three sessions marked
graduation from The Forge as a Junior Security Operator.

---

## Session 16: The Architect's War Room
(OSI Troubleshooting)

### Summary

Session 16 introduced the Outside-In troubleshooting methodology as
a systematic framework for diagnosing failures in complex networked
environments. The session presented live system sabotages — a file
permission lockout and a Docker port conflict — and required students
to identify, localize, and resolve each failure using layer-specific
diagnostic commands. The session reinforced that effective
troubleshooting is not guesswork but a disciplined progression from
the network layer upward through the application layer.

### Key Concepts

**OSI Troubleshooting Methodology:** The OSI model provides a
structured framework for isolating the layer at which a failure
originates. The Outside-In methodology begins at Layer 3 (Network)
and progresses upward: first confirming IP connectivity via `ping`,
then verifying port availability at Layer 4 (Transport) via `nc`,
then inspecting application-layer behavior through logs and service
status, and finally examining filesystem permissions when access
failures are suspected (Tanenbaum & Bos, 2014). This sequence
prevents the common diagnostic error of investigating application
configuration before confirming that the underlying network path
exists.

**Layer 3 Diagnostics:** Layer 3 failures manifest as an inability
to reach a host by IP address. The `ping` command sends ICMP echo
requests to a target and reports whether responses are received,
confirming or ruling out IP-layer connectivity as the source of a
failure. A failed `ping` to a gateway indicates a routing or
interface configuration issue; a successful `ping` to a gateway
but failed `ping` to a remote host indicates a routing issue beyond
the local subnet.

**Layer 4 Diagnostics:** Layer 4 failures manifest as an inability
to establish a TCP or UDP connection to a specific port despite
IP-layer connectivity being intact. The `nc` (netcat) command
attempts a connection to a specified host and port, confirming
whether the target port is open and accepting connections. A Layer
4 failure with Layer 3 connectivity intact indicates either that
the service is not running, that it is bound to a different port,
or that a firewall rule is blocking the connection.

**Layer 7 Diagnostics:** Layer 7 failures manifest as incorrect
application behavior despite network and transport connectivity
being intact. Diagnosis at this layer requires inspection of
application logs via `journalctl` or service-specific log files,
verification of service status via `systemctl`, and examination
of configuration files for syntax errors or misconfigured
parameters.

**File Permission Lockout:** A permission lockout occurs when a
file or directory has been configured with access controls that
prevent the expected user or process from reading, writing, or
executing it. Diagnosis requires `ls -la` to inspect the current
permission assignments and `stat` to examine ownership, followed
by `chmod` or `chown` to restore the expected access configuration.

**Docker Port Conflict:** A Docker port conflict occurs when a
container attempts to bind to a host port that is already in use
by another process. The conflict prevents the container from
starting and manifests as an error in the Docker daemon log.
Diagnosis requires identifying the process occupying the conflicting
port using `ss -tulnp` and either terminating that process or
reconfiguring the container to bind to an available port.

### Tools Used

| Tool | Purpose |
|------|---------|
| `ping` | Verify Layer 3 IP connectivity |
| `nc` (netcat) | Verify Layer 4 port availability |
| `ls -la` | Inspect file and directory permission assignments |
| `systemctl status` | Check service running state |
| `journalctl -xe` | Inspect system and service logs |
| `ss -tulnp` | Identify processes bound to specific ports |
| `chmod` / `chown` | Restore correct file permission configuration |
| `docker ps -a` | Identify failed or conflicting containers |

### Lab Observations

The Break/Fix Gauntlet micro-lab ran a provisioning script that
introduced two simultaneous sabotages: a critical directory was
set to permission mode `000`, blocking all access, and a Docker
container was configured to bind to port 80 which was already
occupied by a host process. The Outside-In methodology was applied
sequentially — `ping` confirmed network layer integrity, `nc`
identified the port conflict at Layer 4, `ls -la` revealed the
permission lockout, and `systemctl` confirmed the affected service
was stopped. Both issues were resolved — the permission restored
via `chmod` and the port conflict resolved by identifying and
stopping the conflicting process with `ss -tulnp` — and findings
were documented in `readiness_check.log`.

### Connection to Defensive Practice

The Outside-In diagnostic methodology is directly transferable to
incident response triage. When a production system becomes
unreachable or behaves unexpectedly, the ability to systematically
rule out each OSI layer as the source of failure dramatically
reduces mean time to resolution. In a security context, the same
methodology applies to investigating potential network-based attacks:
confirming whether a suspected C2 connection exists at Layer 3,
whether a suspicious port is open at Layer 4, and whether anomalous
application behavior is visible at Layer 7 provides a structured
evidence chain for incident documentation.

---

## Session 17: The Forge Final
(Technical Diagnostic)

### Summary

Session 17 assessed Sprint 1 competency across all five domains
through a two-part examination conducted without instructional
guidance or AI assistance. Part A consisted of a timed theory quiz
covering OSI layer identification, subnetting mathematics, Active
Directory hierarchy, and Python logic. Part B was a practical
examination requiring students to locate hidden root-owned log
files on the virtual machine, relocate them to a specified
directory, and apply precise permission configurations to lock
them down.

### Key Concepts

**Examination Under Pressure:** Professional security operations
require the ability to execute diagnostic and remediation procedures
correctly under time constraints and without reference to
documentation. The timed examination format simulated the conditions
of a real incident response scenario, in which the cost of
hesitation or error is measured in system availability and data
integrity rather than exam points.

**Hidden File Discovery:** Files prefixed with a period are hidden
from standard `ls` output in Linux and are only visible when the
`-a` flag is used. Root-owned files in unexpected locations are a
common indicator of attacker activity — an adversary who has
achieved root access may plant files in obscure locations to
establish persistence or stage data for exfiltration. Proficiency
with `find` and `ls -la` is therefore both an examination skill
and a threat hunting skill.

**Permission Lockdown:** Applying precise permissions to sensitive
log files prevents unauthorized modification or deletion of forensic
evidence. In an incident response context, log files represent the
evidentiary record of attacker activity and must be protected from
tampering. Configuring log files as root-owned and read-only for
all other users ensures that non-privileged processes and accounts
cannot alter the record.

### Tools Used

| Tool | Purpose |
|------|---------|
| `find / -name "*.log"` | Locate hidden log files across the filesystem |
| `ls -la` | Confirm file ownership and permission assignments |
| `mv` | Relocate files to the specified destination directory |
| `chmod` | Apply precise permission configuration to log files |
| `chown` | Verify and correct file ownership assignments |

### Lab Observations

Part A of the examination tested theoretical knowledge across OSI
layer identification, binary subnetting calculations, Active
Directory structural hierarchy, and Python control flow logic.
Part B required using `find` with appropriate flags to locate
hidden root-owned `.log` files distributed across the filesystem,
moving them to a designated directory using `mv`, and applying
`chmod 400` to make them read-only for the owner with no access
for group or others. Findings and command output were compiled
into `practical_exam_report.txt` as the session artifact.

### Connection to Defensive Practice

The practical examination directly simulated threat hunting and
forensic preservation workflows. Locating hidden files, auditing
their ownership, and applying protective permissions are actions
performed during active incident response when an analyst is
attempting to identify attacker-planted files and preserve existing
forensic evidence. The ability to execute these operations correctly
without a guide reflects the operational readiness expected of a
Junior Security Operator entering a SOC environment.

---

## Session 18: The Capstone — The Hardened Outpost

### Summary

Session 18 was a solo, three-hour capstone deployment requiring
the construction of a fully integrated, cross-platform enterprise
environment for a fictional client, Titan Small Business Services.
The build required promoting a Windows Server to a Domain Controller,
joining a Linux server to the Active Directory domain, writing a
Python auditing script, and architecting an air-gapped Docker
Compose stack. All work was documented in a professional Security
Architecture Document compiled as a PDF artifact.

### Key Concepts

**Security Architecture Document (SAD):** A Security Architecture
Document is a formal deliverable that describes the design,
configuration, and security rationale of a deployed environment.
A professional SAD includes a system overview, network topology,
identity architecture, policy configuration, and operational
procedures sufficient for a successor administrator to understand
and maintain the environment (NIST, 2018). Writing a SAD in
Markdown and compiling it to PDF is a standard practice in
consulting and enterprise security engineering contexts.

**Full-Stack Integration:** The capstone environment required four
distinct technology domains to function as an integrated whole:
Windows Server Active Directory for identity, Linux domain
integration for cross-platform authentication, Python scripting
for automated auditing, and Docker Compose for containerized
application hosting with network segmentation. The integration
test for each component was not whether it functioned in isolation
but whether it functioned correctly as part of the complete
architecture.

**Solo Deployment Under Time Constraint:** Professional security
deployments occur under time and resource constraints that require
practitioners to prioritize, sequence, and execute tasks without
waiting for guidance. The three-hour capstone window required
strategic decision-making about build order — establishing the
Domain Controller before attempting domain joins, confirming
network connectivity before deploying containerized services —
reflecting the sequencing discipline required in real deployment
scenarios.

### Tools Used

| Tool | Purpose |
|------|---------|
| Windows Server Manager | Promote server to Domain Controller |
| `realm join` | Join Linux server to Active Directory domain |
| Python | Write automated auditing script |
| `docker-compose up` | Deploy air-gapped multi-container stack |
| Markdown / `pandoc` | Author and compile Security Architecture Document |

### Lab Observations

The Hardened Outpost capstone was executed in four sequential
phases. The Windows Server was first promoted to a Domain Controller
for the `titan.local` forest, establishing the identity foundation
for the environment. The Linux server was then joined to the domain
using `realmd` and SSSD, with the sudoers bridge configured to
grant Domain Admins root-level access. A Python auditing script
was written to inspect running services and log their status to
a structured output file. Finally, a Docker Compose stack was
deployed with a segmented network architecture isolating the
application tier from the database tier using an internal bridge
network. All components and their security rationale were
documented in `HardenedOutpost_SAD.pdf`, the capstone artifact.

### Connection to Defensive Practice

The Hardened Outpost capstone represented the practical synthesis
of The Forge curriculum: the ability to design, deploy, and
document a secure enterprise environment without assistance. In
professional contexts, this capability corresponds to the
responsibilities of a security engineer or systems architect
tasked with standing up infrastructure for a new client or
internal project. The Security Architecture Document produced
during the capstone is the type of deliverable expected in
consulting engagements, compliance audits, and internal security
reviews, making its production a directly transferable professional
skill.

---

## References

NIST. (2018). *Framework for improving critical infrastructure
    cybersecurity (version 1.1)*.
    https://nvlpubs.nist.gov/nistpubs/CSWP/NIST.CSWP.04162018.pdf

Tanenbaum, A. S., & Bos, H. (2014). *Modern operating systems*
    (4th ed.). Pearson.
