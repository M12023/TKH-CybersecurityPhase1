# Week 08 — Session Notes
**Operator:** Maurice Ratiff III
**Topic:** The Breach — Exploitation, Privilege Escalation, and
Post-Exploitation
**Date:** May 29th 2026

---

## Summary

Week 8 advanced from reconnaissance and vulnerability identification
into the exploitation and post-exploitation phases of the Cyber
Attack Lifecycle. Building directly on the target profiles and
vulnerability assessments produced in Week 7, this week's sessions
covered live exploitation using the Metasploit Framework, privilege
escalation on both Linux and Windows platforms, persistence
establishment through cron-based scheduling, and lateral movement
through network pivoting. The Take-Home Lab shifted perspective back
to the defensive side, requiring the construction of a Python-based
brute-force detection tool. The week reinforced the foundational
principle that offensive fluency is prerequisite to building
defenses capable of detecting and containing each phase of an
attack chain.

---

## Session 22: The Verification Protocol
(Exploitation & Metasploit)

### Summary

Session 22 established that identifying a vulnerability has no
operational value without the ability to prove exploitation.
The session introduced reverse shell mechanics using Netcat as a
foundational concept, then advanced to the Metasploit Framework
for structured exploit delivery against a legacy Windows target
running the EternalBlue vulnerability — one of the most
consequential vulnerabilities in modern cybersecurity history.

### Key Concepts

**Reverse Shell:** A reverse shell is a network connection initiated
outbound from the target machine to the attacker's listener, rather
than inbound from the attacker to the target. This connection
direction is significant because most firewall configurations permit
outbound connections while blocking unsolicited inbound connections,
making reverse shells the preferred post-exploitation communication
mechanism. Netcat (`nc`) can establish a basic reverse shell by
redirecting standard input, output, and error streams over a TCP
connection (Engebretson, 2013).

**Metasploit Framework:** The Metasploit Framework is an open-source
penetration testing platform that provides a structured environment
for exploit development, delivery, and post-exploitation operations.
Metasploit organizes its functionality into modules including
exploits, payloads, auxiliaries, and post-exploitation tools, all
accessible through a unified command interface called `msfconsole`
(Kennedy et al., 2011). Metasploit is the industry standard tool
for structured exploitation in both offensive security assessments
and security research.

**EternalBlue (MS17-010):** EternalBlue is a critical vulnerability
in the Windows Server Message Block (SMB) protocol that allows
remote code execution without authentication on unpatched Windows
systems. Originally developed by the NSA and leaked publicly in
2017, EternalBlue was subsequently weaponized in the WannaCry and
NotPetya ransomware campaigns, making it one of the most damaging
vulnerabilities in cybersecurity history (MITRE, 2023). Its
inclusion in the Metasploit Framework as the `exploit/windows/smb/
ms17_010_eternalblue` module makes it a standard reference exploit
for training environments.

**Meterpreter:** Meterpreter is an advanced Metasploit payload that
provides an interactive shell with extensive post-exploitation
capabilities including file system access, process migration, network
pivoting, and credential harvesting. Unlike a basic reverse shell,
Meterpreter operates entirely in memory without writing files to
disk, reducing its forensic footprint on the target system.

### Tools Used

| Tool | Purpose |
|------|---------|
| `nc -lvnp` | Start a Netcat listener to catch a reverse shell |
| `msfconsole` | Launch the Metasploit Framework console |
| `use exploit/windows/smb/ms17_010_eternalblue` | Load EternalBlue module |
| `set RHOSTS` | Configure the target IP address |
| `set PAYLOAD` | Select the Meterpreter payload |
| `run` | Execute the exploit module |
| `sessions -i` | Interact with an established Meterpreter session |

### Lab Observations

The session began with a manual reverse shell exercise using Netcat,
establishing a listener on the attacker machine and triggering a
connection from the target, confirming the foundational mechanics
of outbound shell communication. The Metasploit exercise loaded the
EternalBlue module, configured the target host address, set the
Windows Meterpreter payload, and executed the exploit, resulting
in a Meterpreter session on the target system. Post-exploitation
commands confirmed the session was running with SYSTEM-level
privileges, and the successful exploitation was documented in
`exploit_verification.png` as the session artifact.

### Connection to Defensive Practice

Understanding the EternalBlue exploit chain from an attacker's
perspective is directly applicable to defensive detection. The
exploit generates distinctive SMB traffic patterns, produces
specific Windows Event Log entries, and executes a memory-resident
payload that leaves behavioral indicators detectable by endpoint
detection and response tools. A defender who has executed this
attack chain understands which log sources to monitor, which network
signatures to alert on, and why patching MS17-010 remains a
critical remediation item on any network that still runs legacy
Windows systems. The lesson that an unpatched SMB vulnerability
can yield SYSTEM access without authentication underscores the
operational urgency of patch management programs.

---

## Session 23: Climbing the Ladder
(Privilege Escalation)

### Summary

Session 23 addressed the reality that initial exploitation rarely
yields the highest level of system privilege, and that escalating
from a restricted foothold to full administrative control is a
distinct phase of the attack lifecycle requiring its own techniques.
The session covered Linux privilege escalation through sudo
misconfiguration using the GTFOBins database, and Windows privilege
escalation through an unquoted service path vulnerability identified
by WinPEAS.

### Key Concepts

**Privilege Escalation:** Privilege escalation is the process of
obtaining higher system privileges than those initially granted
following initial access. Vertical privilege escalation involves
moving from a standard user account to an administrative or root
account. Horizontal privilege escalation involves moving from one
user account to another account with different or additional access.
Both forms are standard phases of the attack lifecycle following
initial exploitation (MITRE, 2023).

**GTFOBins:** GTFOBins is a curated database of Unix binaries that
can be exploited by attackers who have obtained limited execution
capability to escalate privileges, escape restricted environments,
or establish persistence. When a standard user is granted sudo
permission to run a binary that GTFOBins identifies as exploitable
— such as `vim`, `find`, or `awk` — that user can leverage the
binary's built-in functionality to spawn a privileged shell,
bypassing the intended restriction of their account (GTFOBins,
2023).

**Sudo Misconfiguration:** The `/etc/sudoers` file defines which
users may run which commands with elevated privileges on a Linux
system. A misconfigured sudoers entry that grants a user the ability
to run an exploitable binary without a password represents a direct
privilege escalation path. This class of misconfiguration is a
common finding in Linux security audits and is among the first
checks performed by automated privilege escalation tools.

**Unquoted Service Path:** On Windows systems, when a service's
executable path contains spaces and is not enclosed in quotation
marks, the Windows service manager attempts to resolve the path
by testing multiple interpretations. An attacker who can write to
any directory in the unquoted path can place a malicious executable
that Windows will execute as the service starts, with the privileges
of the service account — frequently SYSTEM (Kennedy et al., 2011).

**WinPEAS:** WinPEAS (Windows Privilege Escalation Awesome Scripts)
is an automated enumeration tool that scans a Windows system for
common privilege escalation vectors including unquoted service
paths, weak service permissions, stored credentials, and
misconfigured registry keys. WinPEAS output provides a prioritized
list of escalation opportunities on the target system.

### Tools Used

| Tool | Purpose |
|------|---------|
| `sudo -l` | List sudo permissions for the current user |
| GTFOBins | Reference database for exploitable sudo binaries |
| WinPEAS | Automated Windows privilege escalation enumeration |
| `sc qc` | Query Windows service configuration |
| Metasploit `service_hijack` | Exploit unquoted service path |

### Lab Observations

The Linux escalation exercise identified a sudo permission granting
the current user the ability to run a specific binary without a
password. Cross-referencing the binary against GTFOBins revealed
a documented escalation technique, which was executed to spawn a
root shell. The Windows escalation exercise ran WinPEAS against
the target system, which identified a service configured with an
unquoted executable path containing spaces. A malicious payload
was placed in the exploitable directory location, and the service
was restarted, executing the payload with SYSTEM privileges. Both
escalation paths and their remediation recommendations were
documented in `escalation_path.txt`.

### Connection to Defensive Practice

Privilege escalation detection is a critical component of endpoint
security monitoring. Both escalation techniques covered in this
session leave detectable indicators: sudo abuse generates entries
in `/var/log/auth.log`, and unquoted service path exploitation
produces Windows Event ID 7045 (new service installed) followed by
anomalous service execution behavior. Defensive hardening against
these vectors is straightforward — regular sudo permission audits,
GTFOBins cross-referencing during access reviews, and automated
scanning for unquoted service paths using tools such as PowerSploit's
`Get-ServiceUnquoted` are standard components of Windows and Linux
hardening checklists.

---

## Session 24: The Deep Network
(Persistence & Pivoting)

### Summary

Session 24 addressed the post-exploitation objectives of persistence
and lateral movement — the techniques that allow an attacker to
maintain access through system reboots and extend their reach into
network segments not directly accessible from the initial point of
compromise. The session covered cron-based backdoor installation
on Linux and network pivoting through a compromised host using
Metasploit autoroute and proxychains.

### Key Concepts

**Persistence:** Persistence refers to techniques that allow an
attacker to maintain access to a compromised system across
interruptions such as reboots, credential changes, or session
timeouts. Establishing persistence transforms a temporary foothold
into a durable presence that survives the disruptions most likely
to terminate an active session (MITRE, 2023).

**Cron-Based Backdoor:** The Linux cron scheduler executes commands
at defined time intervals. A cron entry that runs a reverse shell
command every minute will automatically re-establish an attacker's
connection each time the previous session is terminated, including
after system reboots if placed in the system-wide crontab. This
persistence mechanism requires no additional software installation,
leaving a minimal footprint beyond the crontab modification itself.

**Network Pivoting:** Network pivoting is the technique of using
a compromised host as a relay to attack systems on network segments
that are not directly accessible from the attacker's machine. The
compromised host has legitimate network access to the target subnet;
the attacker routes their traffic through the compromised host's
network stack to reach otherwise unreachable systems (Engebretson,
2013).

**Metasploit Autoroute:** The Metasploit `autoroute` post-
exploitation module adds a route to Metasploit's internal routing
table directing traffic for a specified subnet through an active
Meterpreter session. This enables other Metasploit modules to
reach hosts on the target subnet without any configuration changes
on the compromised host itself, as the routing occurs entirely
within Metasploit's session management layer.

**SOCKS Proxy and Proxychains:** Metasploit's `auxiliary/server/
socks_proxy` module starts a SOCKS proxy server that accepts
connections on the attacker's machine and forwards them through
the Metasploit routing table into the target network. Proxychains
forces external tools — including Nmap — to route their connections
through this SOCKS proxy, enabling arbitrary tools to reach hosts
on network segments that Metasploit has routed through a pivot
session.

### Tools Used

| Tool | Purpose |
|------|---------|
| `crontab -e` | Edit cron schedule to install persistent backdoor |
| `nc -lvnp` | Listen for reverse shell connections |
| Metasploit `autoroute` | Add route to private subnet through Meterpreter session |
| `auxiliary/server/socks_proxy` | Start SOCKS proxy for external tool tunneling |
| `proxychains nmap` | Scan private subnet through the SOCKS tunnel |

### Lab Observations

The persistence exercise established an SSH session on the
compromised web server, added a cron entry executing a reverse
shell command every minute targeting the attacker's Netcat listener,
and confirmed that a fresh shell spawned automatically within 60
seconds of terminating the previous session. The pivoting exercise
established a Meterpreter session on the web server, ran autoroute
to add a route for the `10.0.9.0/24` private subnet, backgrounded
the session, and started the SOCKS proxy module on port 1080.
Proxychains was then used to force an Nmap TCP connect scan through
the tunnel, successfully enumerating open ports on the database
host at `10.0.9.50` — a system with no direct network path from
the attacker's machine. The successful scan output was captured
as `pivot_success.png`.

### Connection to Defensive Practice

Cron-based persistence and network pivoting both operate through
legitimate system mechanisms, making them harder to detect than
novel malware. Cron backdoor detection requires monitoring for
unexpected modifications to crontab files — specifically `auditd`
rules targeting crontab write events and `/var/spool/cron/`
directory changes. Pivot detection requires east-west traffic
inspection between network segments: traffic flowing from a DMZ
web server to a backend database server on arbitrary ports, or
at volumes inconsistent with normal application behavior, is a
strong indicator of pivoting activity. Network segmentation enforced
at the firewall level — restricting the web server to only the
ports and protocols required to reach the database — limits the
utility of an established pivot by constraining what the attacker
can reach even after compromising the pivot host.

---

## TLAB 8: The Paper Trail
(Brute-Force Detector)

### Summary

The Week 8 Take-Home Lab shifted to the defensive perspective,
requiring the construction of a Python-based brute-force detection
tool. The script read a simulated authentication log, filtered for
failed login attempt entries, counted failure occurrences per source,
and wrote a structured forensic report identifying sources that
exceeded a defined failure threshold. The artifact `brute_detector.py`
demonstrated the practical application of the log parsing and file
I/O skills developed in Week 3 to a real defensive use case.

### Connection to Defensive Practice

Brute-force attack detection is a standard SOC function implemented
in every mature security monitoring environment. The Python
implementation produced in this lab mirrors the underlying logic
of commercial SIEM correlation rules that alert on authentication
failure thresholds. Building this detection capability from scratch
provides direct insight into the data sources, parsing logic, and
threshold tuning decisions that determine whether a brute-force
attack is detected in near-real-time or discovered days later
during a post-incident review.

---

## References

Engebretson, P. (2013). *The basics of hacking and penetration
    testing: Ethical hacking and penetration testing made easy*
    (2nd ed.). Syngress.

GTFOBins. (2023). *GTFOBins: Unix binaries that can be used to
    bypass local security restrictions*.
    https://gtfobins.github.io/

Kennedy, D., O'Gorman, J., Kearns, D., & Aharoni, M. (2011).
    *Metasploit: The penetration tester's guide*. No Starch Press.

MITRE. (2023). *MITRE ATT&CK: Design and philosophy*.
    https://attack.mitre.org/docs/ATTACK_Design_and_Philosophy_March_2020.pdf
