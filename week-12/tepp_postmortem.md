# Phase 1 Final Reckoning — TEPP Post-Mortem
**Operator:** Maurice Ratiff III
**Date:** May 30, 2026
**Repository:** https://github.com/M12023/TKH-CybersecurityPhase1/edit/main/week-12/tepp_postmortem.md
**TKH Innovation Fellowship 2026 | Phase 1 | Cybersecurity**

---

## Phase 0: Reconnaissance

### Triage Network — 172.100.0.0/24

An Nmap full-port version scan (`nmap -sV -sC -p- 172.100.0.0/24`)
identified four live hosts within the triage network. The primary
host at 172.100.0.1 exposed SSH on port 22 running OpenSSH 9.6p1
alongside two HTTP services on ports 8080 and 8082 running nginx
versions 1.24.0 and 1.29.7 respectively — the presence of two
distinct nginx versions on a single host indicates inconsistent
patch management and the port 8080 service flagged as a potential
open proxy, suggesting it may be configured to forward requests
to internal network resources (Lyon, 2009). The host at
172.100.0.11 exposed Redis 8.6.2 on port 6379 with no
authentication mechanism detected, representing a critical
misconfiguration that allows any network-adjacent client to
connect and issue arbitrary commands including full keyspace
enumeration and configuration modification without credentials
(MITRE, 2023). The host at 172.100.0.12 exposed vsftpd 3.0.2
on port 21, a File Transfer Protocol service that transmits
credentials and data in plaintext and was configured with write
access enabled, a disabled chroot jail, and world-permissive
file creation modes — a compounding misconfiguration chain that
creates an upload and directory traversal risk for any
authenticated user. The host at 172.100.0.13 returned no open
ports across all 65,535 scanned ports, and internal inspection
revealed a world-writable `/tmp` directory and a sole process
running as root with no resource restrictions, indicating a
container deployed without application of the principle of
least privilege.

### Breach Network — 172.80.0.0/24

An Nmap full-port version scan of the 172.80.0.0/24 subnet
identified one live host at 172.80.0.10, exposing SSH on port
22 running OpenSSH 9.6p1 as its sole network service. Internal
inspection of the host's SSH configuration revealed two critical
misconfigurations: `PermitRootLogin yes` and
`PasswordAuthentication yes` were both explicitly enabled,
allowing direct remote authentication to the root account using
a password rather than requiring key-based authentication or
restricting root login entirely (NIST, 2018). The host's
`/etc/shadow` file confirmed that the root account was protected
only by a SHA-512 password hash, making it susceptible to
offline cracking or online brute-force attack given the absence
of any rate-limiting, account lockout, or multi-factor
authentication controls on the SSH service. These findings
directly informed the Phase 2 approach: with no lockout
mechanism and password authentication enabled for the root
account, the SSH service presented a viable brute-force target
against which a credential attack could be executed without
triggering automated defensive responses.

### Exploitation Network — 172.60.0.0/24

An Nmap full-port version scan of the 172.60.0.0/24 subnet
identified one live host at 172.60.0.1, exposing SSH on port
22 running OpenSSH 9.6p1 and HTTP services on ports 8080 and
8082 running nginx 1.24.0 and nginx 1.29.7 respectively. The
SSH host key fingerprints on this host matched those observed
across all previously scanned subnets — ECDSA fingerprint
`cc:98:07:cd:c7:1d:26:9d:10:51:5a:d0:50:30:a4:93` and ED25519
fingerprint `da:50:82:67:40:4a:dd:4e:76:4e:9d:6e:e4:c4:e2:e0`
— confirming that a single underlying host is reachable across
all three network segments simultaneously, representing a
fundamental failure of network segmentation in which a
compromised host provides access to all segments regardless
of their intended isolation (Lyon, 2009). The nginx instance
on port 8080 again presented open proxy indicators, suggesting
the web service may be configured to forward requests to backend
systems not directly reachable from the scanning host, which
informed the Phase 3 exploitation approach targeting the
application layer running on this host. The nginx version
discrepancy between ports 8080 and 8082 indicated inconsistent
patch management and the possibility of additional
vulnerabilities in the older 1.24.0 instance running on the
externally accessible port.

---

## Phase 1: Rapid Triage

### Server 1 — 172.100.0.11 (Redis)

**Vulnerability Identified:**
Redis 8.6.2 was running on port 6379 with no authentication
requirement. Confirmation was performed by connecting via
`redis-cli ping`, which returned `PONG` without any credentials,
and `redis-cli config get requirepass`, which returned an empty
string confirming the absence of password enforcement.

**Remediation Commands:**
docker exec -it broken_server_1 sh
redis-cli config set requirepass "TKH$ecure2026!"

**Before State:**
requirepass = "" (no password set)
redis-cli ping → PONG (unauthenticated access confirmed)

**After State:**
requirepass = "TKH!" (password enforced at runtime)
redis-cli ping → (error) NOAUTH Authentication required

**Analysis:**
An unauthenticated Redis instance exposes the full command set
of the database to any network-adjacent client, including
commands that enumerate all stored keys, overwrite configuration
values, and in certain deployment contexts write arbitrary files
to the host filesystem — a vector documented in multiple
real-world ransomware and cryptomining campaigns targeting
exposed Redis services (MITRE, 2023). In an enterprise
environment, a Redis instance reachable without authentication
on a shared network segment represents an immediate data
exfiltration risk, as the database may cache session tokens,
API keys, or application credentials used by upstream services.
Remediation requires both runtime password enforcement via
`config set requirepass` and a persistent configuration file
update to ensure the control survives container or service
restart.

### Server 2 — 172.100.0.12 (vsftpd)

**Vulnerability Identified:**
The vsftpd 3.0.2 configuration file at `/etc/vsftpd/vsftpd.conf`
contained three compounding misconfigurations confirmed via
`grep write /etc/vsftpd/vsftpd.conf` and
`grep file_open_mode /etc/vsftpd/vsftpd.conf`:
`write_enable=YES` permitted file uploads and modifications,
`allow_writeable_chroot=YES` disabled the chroot jail
restriction, and `file_open_mode=0666` set all uploaded files
to world-readable and world-writable by default.

**Remediation Commands:**
docker exec -it broken_server_2 sh
sed -i 's/write_enable=YES/write_enable=NO/' /etc/vsftpd/vsftpd.conf
sed -i 's/allow_writeable_chroot=YES/allow_writeable_chroot=NO/' 
/etc/vsftpd/vsftpd.conf
sed -i 's/file_open_mode=0666/file_open_mode=0644/' 
/etc/vsftpd/vsftpd.conf

**Before State:**
write_enable=YES
allow_writeable_chroot=YES
file_open_mode=0666

**After State:**
write_enable=NO
allow_writeable_chroot=NO
file_open_mode=0644

**Analysis:**
The combination of enabled write access, a disabled chroot
jail, and world-writable file permissions created a compounding
vulnerability chain in which an authenticated FTP user could
upload malicious files, escape their restricted home directory,
and leave those files modifiable by any process running on the
system (NIST, 2018). In an enterprise environment, FTP should
be treated as a legacy protocol requiring strong justification
for deployment, as all credential and data transmission occurs
in plaintext and is trivially interceptable by any host with
access to the network segment. The corrected configuration
enforces read-only access within the chroot boundary and
applies standard file creation permissions that prevent
unauthorized modification of uploaded content.

### Server 3 — 172.100.0.13 (Alpine)

**Vulnerability Identified:**
The `/tmp` directory was configured with permissions
`drwxrwxrwt` — world-writable with the sticky bit set —
confirmed via `ls -la / | grep tmp` inside the container.
The container's sole running process (`sleep infinity`) was
executing as root with no capability restrictions, confirmed
via `ps aux`, and no SUID binaries were present per
`find / -perm -4000 2>/dev/null`.

**Remediation Commands:**
docker exec -it broken_server_3 sh
chmod 1755 /tmp

**Before State:**
/tmp → drwxrwxrwt (world-writable, sticky bit set)
Sole process: sleep infinity running as root (PID 1)

**After State:**
/tmp → drwxr-xr-t (owner write only, sticky bit retained)

**Analysis:**
A world-writable temporary directory in a container running
as root creates a file tampering and privilege escalation
risk, as any process executing within the container context
can create, overwrite, or place symbolic links in `/tmp` that
may be followed by privileged processes or scripts that
reference temporary file paths without validation (Kerrisk,
2010). In an enterprise container deployment, the principle
of least privilege requires that containers run as non-root
users wherever possible and that shared directories restrict
write access to owning processes to prevent inter-process
file manipulation. The remediation retains the sticky bit —
which prevents users from deleting files they do not own —
while removing world-write access, restoring the intended
access control model for a shared temporary directory.

---

## Phase 2: The Breach

**Cracked Credentials:**
- Username: root123
- Password: admin123

**Forensic Evidence:**
- Exact Timestamp of Successful Login: Accepted password for root from 172.80.0.1
  port 38904 ssh2
- Attacker IP Address:  172.80.0.1

**Engineered iptables Rule:**
iptables -A INPUT -s 172.80.0.1 -j DROP

**SOC Analysis:**
A single iptables rule blocking the attacker's source IP
address represents a reactive and fundamentally insufficient
defensive measure because it addresses only the specific
vector observed in the current incident while leaving the
underlying vulnerability — unauthenticated root SSH access
with password authentication enabled — entirely intact and
exploitable from any other source address (NIST, 2018). In
a real enterprise environment, a SOC would deploy a layered
response alongside the IP block: immediate disabling of
password-based SSH authentication in favor of key-only
authentication, implementation of an automated lockout
policy using `fail2ban` or equivalent to rate-limit
authentication attempts from any source, network-level
restriction of SSH access to approved management IP ranges
using firewall rules rather than reactive blocks, and
deployment of SIEM alerting on authentication failure
thresholds to ensure future brute-force attempts are
detected before a successful login occurs (Chuvakin et al.,
2013). The iptables block addresses the symptom; the SSH
hardening addresses the cause.

---

## Phase 3: Full Spectrum

**Listener Configuration:**
No external listener was required for this phase. The exploit
was executed directly against the vulnerable Flask web
application running on port 8080 using curl as the HTTP
client from the attacker's machine.

**Reverse Shell Payload / Exploit Commands:**

Step 1 — Authentication Bypass:
curl "http://localhost:8080/auth?user=admin'--&pass=anything"

Step 2 — UNION Injection to Extract CEO Salary:
curl -g "http://localhost:8080/directory?search=%25%27%20UNION%20SELECT%20name%2Csalary%20FROM%20employees--"


**Command Injection Explanation:**
The titan_webapp application constructs SQL queries by
directly concatenating user-supplied input into query strings
without sanitization or parameterization, making it vulnerable
to SQL Injection rather than OS command injection. SQL
Injection occurs when attacker-controlled input modifies the
logical structure of a database query — in the authentication
endpoint, submitting `admin'--` as the username causes the
WHERE clause to evaluate as always true by commenting out the
password check entirely, granting access without valid
credentials (OWASP, 2021). In the employee directory endpoint,
a UNION-based injection appends an attacker-controlled SELECT
statement to the original query, allowing retrieval of the
salary column from the employees table — data not intended
to be exposed through the search interface — demonstrating
that SQL Injection can bypass both authentication and
authorization controls simultaneously.

**Forensic Evidence:**
- Process ID (PID): 17895
  (confirmed via `ps aux | grep "python3 app.py"`)
- User-Agent:  curl/8.5.0
  (confirmed via Flask access log:
  `127.0.0.1 - - [30/May/2026 19:53:03]
  "GET /directory?search=%25'%20UNION%20SELECT%20name,
  salary%20FROM%20employees-- HTTP/1.1" 200 -`)

**Extracted Data:**
- Auth bypass confirmed: [+] AUTH BYPASS SUCCESS! Welcome Admin.
- CEO salary extracted: Alice (CEO) — $2,500,000
- Engineering salary extracted: Bob — $120,000


**Lockdown Command:**
sudo iptables -A INPUT -s 127.0.0.1 -p tcp --dport 8080 -j DROP

**Final Analytical Paragraph:**
Executing this attack chain from both sides of the operation
reveals that the most consequential vulnerabilities are not
technical in isolation but architectural: the decision to
enable password authentication for the root account, to
deploy a web application that passes user input to a shell
without sanitization, and to connect a single host across
multiple network segments simultaneously each individually
reduced the effort required to achieve full compromise, and
together they created a kill chain that required no novel
techniques — only the systematic exploitation of preventable
misconfigurations (MITRE, 2023). Playing the role of the
attacker demonstrated that reconnaissance findings translate
directly into exploitation decisions: the open proxy
indicator on port 8080 identified during Phase 0 was the
same service that accepted the command injection payload in
Phase 3, confirming that what defenders dismiss as low-
severity findings are frequently the entry points attackers
prioritize. The single defensive control that would have
stopped this breach entirely is input sanitization at the
application layer: if the web service had validated and
escaped user-supplied input before passing it to the system
shell, the command injection payload would have been treated
as literal data rather than executable code, and no
subsequent access — reverse shell, lateral movement, or
data exfiltration — would have been possible regardless of
the network configuration or authentication policies in
place (OWASP, 2021). This operation reinforces the principle
that application-layer controls are the last line of defense
when network and host controls fail, and that security
engineers must validate input handling as rigorously as
they audit firewall rules and authentication configuration.

---

## References

Chuvakin, A., Schmidt, K., & Phillips, C. (2013). *Logging
    and log management: The authoritative guide to
    understanding the concepts surrounding logging and log
    management*. Syngress.

Kerrisk, M. (2010). *The Linux programming interface: A
    Linux and UNIX system programming handbook*. No Starch
    Press.

Lyon, G. (2009). *Nmap network scanning: The official Nmap
    project guide to network discovery and security
    scanning*. Insecure.com. https://nmap.org/book/

MITRE. (2023). *MITRE ATT&CK: Design and philosophy*.
    https://attack.mitre.org/docs/ATTACK_Design_and_Philosophy_March_2020.pdf

NIST. (2018). *Framework for improving critical
    infrastructure cybersecurity (version 1.1)*.
    https://nvlpubs.nist.gov/nistpubs/CSWP/NIST.CSWP.04162018.pdf

OWASP. (2021). *OWASP Top 10 — 2021*.
    https://owasp.org/Top10/
