# Week 11 — Session Notes
**Operator:** Maurice Ratiff III
**Topic:** The Fortress — Network Defense and Perimeter Hardening
**Date:** May 29th 2026

---

## Summary

Week 11 completed the offensive-to-defensive arc of the curriculum
by applying attacker knowledge to the engineering of layered network
defenses. Where prior weeks established fluency in the techniques
used to breach infrastructure, this week's sessions translated that
fluency into the design and deployment of controls capable of
detecting and containing those same techniques. Across three
sessions, coursework covered firewall architecture and egress
filtering, network intrusion detection and custom signature
engineering, and endpoint detection through process monitoring and
behavioral policy. The cumulative Take-Home Lab, Operation Fortress,
required deploying all three defensive layers as a unified Defense
in Depth architecture. The week established that effective network
defense is not the application of vendor defaults but the deliberate
engineering of controls informed by specific attacker behavior.

---

## Session 31: The Barricade
(UFW, iptables, DMZ Architecture & Egress Filtering)

### Summary

Session 31 established the outermost defensive layer: the network
perimeter. The session covered kernel-level firewall engineering
using iptables, DMZ architecture using Docker subnet segmentation,
and egress filtering to prevent compromised servers from
establishing outbound connections to attacker infrastructure.
The session directly applied knowledge of reverse shell mechanics
and network pivoting from Week 8 to the engineering of controls
designed to neutralize those techniques.

### Key Concepts

**Defense in Depth:** Defense in Depth is a security architecture
principle that deploys multiple independent layers of controls such
that the failure of any single layer does not result in complete
compromise. In network security, Defense in Depth is implemented
through perimeter filtering at the network boundary, segmentation
between network zones, detection on the wire, and monitoring at
the endpoint — each layer designed to catch what the previous layer
misses (NIST, 2018).

**DMZ Architecture:** A Demilitarized Zone (DMZ) is a network
segment positioned between the public internet and the internal
network that hosts services requiring external accessibility —
typically web servers, mail servers, and DNS servers. The DMZ
architecture prevents direct connectivity between external users
and internal systems: a user accessing a web server in the DMZ
cannot reach internal database servers even if the web server is
fully compromised, because no network path exists between the DMZ
and the internal network without traversing a firewall (Tanenbaum
& Bos, 2014). Docker bridge networks with internal isolation
provide an equivalent segmentation model within a containerized
environment.

**iptables:** iptables is the Linux kernel's built-in packet
filtering framework, providing granular control over network traffic
at the kernel level. Rules are organized into chains — INPUT for
inbound traffic, OUTPUT for outbound traffic, and FORWARD for
traffic being routed through the host — and evaluated in sequence
until a matching rule is found. iptables operates at a lower level
than application firewalls and cannot be bypassed by user-space
processes, making it the foundational enforcement mechanism for
Linux host-based firewall policy.

**UFW:** Uncomplicated Firewall (UFW) is a front-end interface for
iptables that simplifies the creation of common firewall rules
through human-readable commands. UFW translates high-level rule
specifications into iptables rules, making kernel-level firewall
management accessible without requiring direct iptables syntax
for routine configurations.

**Egress Filtering:** Egress filtering applies firewall rules to
outbound traffic, restricting which destinations and protocols a
host is permitted to reach. From an offensive perspective, reverse
shells, C2 callbacks, and data exfiltration all require outbound
connectivity from the compromised host to attacker infrastructure.
Egress filtering rules that restrict outbound connections to only
the ports and destinations required for legitimate operation
neutralize these techniques by preventing the compromised host
from establishing the outbound connections the attacker depends on
(Engebretson, 2013).

### Tools Used

| Tool | Purpose |
|------|---------|
| `ufw allow` / `ufw deny` | Configure host firewall rules via UFW |
| `iptables -A OUTPUT` | Append egress filtering rules to OUTPUT chain |
| `iptables -L -v` | List active iptables rules with packet counts |
| `docker network create --internal` | Create air-gapped Docker subnet for DMZ |
| `ping` / `nc` | Verify that firewall rules block expected traffic |
| `iptables-save` | Persist firewall rules across reboots |

### Lab Observations

The DMZ architecture exercise created two Docker bridge networks —
a FrontEnd network connecting the web server to the external
interface, and an internal BackEnd network connecting the web server
to the database with no external routing. Connectivity testing
confirmed that the database container was unreachable from outside
the Docker environment while remaining accessible from the web
server container through the internal bridge. The egress filtering
exercise added iptables OUTPUT chain rules restricting the web
server container to outbound connections on only ports 80 and 443,
then attempted to establish a reverse shell callback on port 4444,
confirming that the egress rule blocked the connection. All rule
configurations and test results were documented as the session
artifact.

### Connection to Defensive Practice

Egress filtering is among the highest-value, lowest-cost defensive
controls available to network defenders, yet it is consistently
underimplemented in enterprise environments that focus firewall
policy on inbound traffic. An attacker who has compromised a host
behind a well-configured ingress firewall can frequently still
establish outbound C2 connectivity on common ports such as 80,
443, or 53 because most organizations do not restrict outbound
traffic from servers. Implementing egress filtering that restricts
servers to only the outbound connections required for their
legitimate function eliminates the reverse shell and C2 callback
techniques that are the operational foundation of post-exploitation
activity.

---

## Session 32: The Tripwire
(Suricata IDS & Custom Signature Engineering)

### Summary

Session 32 deployed Suricata as a network intrusion detection sensor
and introduced the engineering of custom detection signatures. The
session covered Suricata rule syntax, signature logic, and the
interpretation of alert output, culminating in the creation of a
custom rule capable of detecting a specific malicious payload
crossing the network wire — translating attacker payload knowledge
from prior weeks directly into detection capability.

### Key Concepts

**Intrusion Detection System (IDS):** A Network Intrusion Detection
System (NIDS) monitors network traffic passively, analyzing packets
against a library of signatures and behavioral rules to identify
traffic matching known attack patterns. Unlike a firewall, which
enforces access control by blocking traffic, an IDS generates alerts
when suspicious traffic is observed, providing visibility into
attack activity without disrupting network flow (Bejtlich, 2004).

**Suricata:** Suricata is an open-source, high-performance network
threat detection engine capable of operating as an IDS, Intrusion
Prevention System (IPS), and network security monitoring platform.
Suricata supports multi-threading for high-throughput environments
and can analyze protocols at the application layer, enabling
detection of attacks embedded within legitimate protocol traffic
(OISF, 2023).

**Suricata Rule Syntax:** A Suricata rule consists of an action,
a header specifying the traffic direction and addressing, and a
set of options defining what to match and how to alert. The action
field specifies whether to alert, drop, reject, or pass matching
traffic. The header defines the protocol, source and destination
addresses, and port ranges. Options include content matching for
specific byte sequences, protocol field matching, flow direction
specification, and alert metadata including signature ID, message,
and severity classification.

**Custom Signature Engineering:** Default IDS signature libraries
detect known, publicly documented attack patterns. Custom signatures
extend detection coverage to organization-specific threats,
attacker tooling identified during threat intelligence analysis,
and attack patterns for which no public signature exists. Effective
custom signature engineering requires knowledge of the specific
payload, protocol, or behavioral characteristic that distinguishes
the target traffic from benign traffic, making attacker knowledge
directly applicable to detection capability development.

**`fast.log`:** Suricata's `fast.log` file records a single-line
summary of each triggered alert, including the timestamp, signature
message, source and destination addresses, and protocol. Monitoring
`fast.log` in real time using `tail -f` provides an immediate view
of detection events as they occur, enabling rapid confirmation that
a deployed signature is triggering correctly.

### Tools Used

| Tool | Purpose |
|------|---------|
| `suricata -c` | Launch Suricata with specified configuration |
| `/etc/suricata/rules/` | Directory for custom signature files |
| `tail -f fast.log` | Monitor real-time alert output |
| `suricata-update` | Update community signature library |
| `tcpreplay` | Replay captured traffic to test signatures |

### Lab Observations

The session deployed Suricata in IDS mode on the Ubuntu VM,
configured to monitor the network interface connected to the Docker
target network. The community rule library was updated and Suricata
launched, with `tail -f /var/log/suricata/fast.log` providing real-
time alert visibility. A custom signature was engineered to detect
a specific string payload associated with the reverse shell technique
covered in Week 8, using the `content` option to match the payload
byte sequence and the `flow` option to restrict matching to the
appropriate traffic direction. The signature was added to a custom
rules file, Suricata was reloaded, and the trigger traffic was
generated, producing an alert entry in `fast.log` confirming
successful detection. The custom rule and alert output were
documented as the session artifact.

### Connection to Defensive Practice

Custom signature engineering is the capability that separates
reactive security operations — waiting for a vendor to release a
signature — from proactive threat-informed defense. When threat
intelligence identifies a specific attacker tool, payload, or
technique, the ability to translate that intelligence into a
detection signature and deploy it within the environment
immediately, before a vendor rule is available, is a significant
defensive advantage. The reverse shell payload signature engineered
in this session was derived directly from the offensive technique
practiced in Week 8, demonstrating the direct translation path
from offensive knowledge to defensive capability that is the
foundational principle of threat-informed defense.

---

## Session 33: The Last Mile
(SysmonForLinux, Process Tracking & Ransomware Detection)

### Summary

Session 33 addressed the innermost defensive layer: endpoint
detection and response. The session deployed Microsoft's
SysmonForLinux to enable granular process monitoring on the Ubuntu
endpoint and introduced XML-based detection policy configuration
for identifying ransomware precursor behaviors before encryption
activity begins. The session established that network-layer
detection is insufficient against attackers who operate entirely
within a compromised host's legitimate process space.

### Key Concepts

**Endpoint Detection and Response (EDR):** EDR refers to the class
of security tools that monitor endpoint activity at the process
and system call level, recording process creation, network
connections, file system modifications, and inter-process
communication. EDR data provides visibility into attacker behavior
that occurs entirely within a host and generates no distinctive
network traffic, making it complementary to network-layer detection
rather than a substitute for it (MITRE, 2023).

**SysmonForLinux:** Sysmon (System Monitor) is a Windows Sysinternals
tool ported to Linux by Microsoft that logs granular system activity
to a structured event log. On Linux, SysmonForLinux captures process
creation events including the full command line, network connection
events, and file creation events, providing the high-fidelity
endpoint telemetry required for behavioral detection and forensic
investigation (Microsoft, 2023).

**Process Creation Monitoring:** Recording every process creation
event with its full command line, parent process, and execution
context provides a behavioral audit trail of all activity on the
endpoint. Attacker tools that execute without writing files to disk
— such as Meterpreter — still generate process creation events
when launching post-exploitation commands, making process monitoring
a detection layer that persists even against fileless attack
techniques.

**XML Detection Policy:** SysmonForLinux is configured through an
XML policy file that defines which events to capture and which to
filter. Detection rules within the policy specify process names,
command-line patterns, file path patterns, and network destinations
that should generate alert events. An XML policy targeting
ransomware precursor behaviors — such as mass file enumeration,
shadow copy deletion commands, or execution of known encryption
tool signatures — can detect ransomware activity in the
reconnaissance and staging phases before encryption begins.

**Ransomware Precursor Detection:** Ransomware attacks follow a
consistent behavioral pattern before encryption begins: the malware
enumerates file system contents to identify encryption targets,
deletes Volume Shadow Copies to prevent recovery, and may establish
persistence before initiating the encryption routine. Each of these
precursor behaviors generates detectable process and file system
events that can be matched by EDR policy rules, enabling detection
and containment before data loss occurs (MITRE, 2023).

### Tools Used

| Tool | Purpose |
|------|---------|
| `sysmon -i` | Install SysmonForLinux with configuration policy |
| XML configuration file | Define process and file event detection rules |
| `journalctl -f` | Monitor SysmonForLinux event output in real time |
| `pwsh` (PowerShell Core) | Execute detection policy management commands |
| `ausearch` | Query audit log for specific event types |

### Lab Observations

The session installed SysmonForLinux and loaded a baseline
configuration policy. An XML detection rule was written targeting
command-line patterns associated with ransomware precursor activity,
including `vssadmin delete shadows`, mass `find` commands against
the home directory tree, and execution of base64-encoded commands
— a common obfuscation technique used by ransomware loaders.
Simulated precursor commands were executed on the endpoint and
`journalctl -f` confirmed that SysmonForLinux generated alert events
matching each rule. The detection policy demonstrated that the
simulated ransomware preparation activity would have been flagged
before any encryption occurred, providing the response window
required for containment. The XML policy and event log output were
documented as the session artifact.

### Connection to Defensive Practice

Ransomware is the most financially damaging category of cyber attack
affecting organizations across all sectors, with average incident
costs measured in millions of dollars in recovery expenses,
regulatory penalties, and reputational damage (NIST, 2018).
Signature-based detection of ransomware executables is frequently
ineffective because ransomware authors modify their binaries to
evade known signatures. Behavioral detection through process
monitoring — alerting on what the ransomware does rather than what
it is — provides a detection mechanism that remains effective
regardless of binary modification. The XML EDR policy developed
in this session targets behaviors that are structurally necessary
for ransomware to function, making evasion significantly more
difficult than evading file-based signatures.

---

## TLAB 11: Operation Fortress
(Unified Defense in Depth Deployment)

### Summary

The Week 11 Take-Home Lab required deploying all three defensive
layers from the week's sessions as a unified, integrated Defense
in Depth architecture. Operation Fortress combined egress firewall
engineering to block reverse shell callbacks (Session 31), custom
Suricata signatures to detect malicious payload patterns on the
wire (Session 32), and an XML EDR policy to catch ransomware
precursor behavior at the endpoint (Session 33). Each layer was
tested independently and then validated as an integrated stack,
confirming that the architecture would detect and contain attacker
activity at the perimeter, network, and endpoint levels
simultaneously.

### Connection to Defensive Practice

Operation Fortress demonstrated the operational principle that
Defense in Depth is not the deployment of three independent tools
but the engineering of three coordinated layers that address
different attacker techniques and compensate for each other's
blind spots. An attacker who bypasses egress filtering by tunneling
C2 traffic over HTTPS on port 443 may still be detected by a
Suricata signature matching the payload pattern within the encrypted
tunnel's metadata. An attacker who evades network detection
entirely by operating in memory may still be detected by SysmonForLinux
process creation events. The layered architecture ensures that
an attacker must evade all three layers simultaneously to operate
undetected — a significantly higher bar than defeating any single
control.

---

## References

Bejtlich, R. (2004). *The Tao of network security monitoring:
    Beyond intrusion detection*. Addison-Wesley.

Engebretson, P. (2013). *The basics of hacking and penetration
    testing: Ethical hacking and penetration testing made easy*
    (2nd ed.). Syngress.

Microsoft. (2023). *Sysmon for Linux*.
    https://github.com/Sysinternals/SysmonForLinux

MITRE. (2023). *MITRE ATT&CK: Design and philosophy*.
    https://attack.mitre.org/docs/ATTACK_Design_and_Philosophy_March_2020.pdf

NIST. (2018). *Framework for improving critical infrastructure
    cybersecurity (version 1.1)*.
    https://nvlpubs.nist.gov/nistpubs/CSWP/NIST.CSWP.04162018.pdf

OISF. (2023). *Suricata user guide*.
    https://suricata.readthedocs.io/

Tanenbaum, A. S., & Bos, H. (2014). *Modern operating systems*
    (4th ed.). Pearson.
