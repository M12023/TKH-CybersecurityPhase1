# Week 10 — Session Notes
**Operator:** Maurice Ratiff III
**Topic:** The Defender — Digital Forensics and Incident Response
(DFIR)
**Date:** May 29th 2026

---

## Summary

Week 10 marked the transition from offensive operator to incident
responder and forensic analyst, applying the attacker knowledge
developed across Sprint 2 to the discipline of Digital Forensics
and Incident Response. Across three sessions, coursework covered
the six-stage IR lifecycle, live system triage and evidence
preservation, cryptographic hashing for chain of custody, memory
forensics and disk imaging, deleted file recovery, and SIEM-based
log correlation for attack timeline reconstruction. The cumulative
Take-Home Lab, Operation Phantom Pursuit, required integrating all
three skill sets into a complete forensic investigation. The week
established that effective incident response is not reactive
improvisation but a disciplined, evidence-driven process grounded
in the same technical knowledge used to conduct the attacks being
investigated.

---

## Session 28: The Crime Scene
(Live Response, Hashing & Chain of Custody)

### Summary

Session 28 introduced the foundational principles and procedures
of incident response, establishing the PICERL lifecycle as the
professional framework for managing security incidents from
detection through lessons learned. The session covered live system
triage methodology, the collection of volatile evidence before
system shutdown, and the use of cryptographic hashing to establish
and maintain chain of custody for forensic evidence.

### Key Concepts

**PICERL Incident Response Lifecycle:** The PICERL model defines
six stages of a structured incident response process: Preparation,
Identification, Containment, Eradication, Recovery, and Lessons
Learned. Preparation encompasses the policies, tools, and trained
personnel required before an incident occurs. Identification is
the process of detecting and confirming that a security incident
has taken place. Containment limits the spread and impact of the
incident. Eradication removes the attacker's presence and the
vulnerability that enabled access. Recovery restores affected
systems to normal operation. Lessons Learned documents findings
and improves defenses against recurrence (NIST, 2012).

**Volatile Evidence:** Volatile evidence refers to data that exists
only in active system memory and is permanently lost when a system
is powered off. This includes running processes, active network
connections, logged-in users, open file handles, and decrypted
content that may be encrypted on disk. Live response — the practice
of collecting volatile evidence from a running system before
shutdown — is the first priority of an incident responder arriving
at a compromised host, as the order of volatility dictates that
the most ephemeral data must be captured first (Carrier, 2005).

**Order of Volatility:** The order of volatility is a framework
for prioritizing evidence collection based on how quickly data
changes or disappears. From most to least volatile: CPU registers
and cache, system memory (RAM), network state and connections,
running processes, disk storage, and archival media. An incident
responder who shuts down a system before collecting memory evidence
destroys the most volatile and frequently most revealing forensic
data available.

**Cryptographic Hashing:** A cryptographic hash function produces
a fixed-length digest from an input of arbitrary length. For
forensic purposes, MD5 and SHA256 hashes of evidence files serve
as digital fingerprints — if the hash of an evidence file computed
at collection matches the hash computed at analysis, the file has
not been modified. This mathematical verification is the technical
foundation of chain of custody in digital forensics (Casey, 2011).

**Chain of Custody:** Chain of custody is the documented record
of who collected evidence, when it was collected, how it was stored,
and who has accessed it at each subsequent stage. In legal
proceedings, an unbroken chain of custody is required for digital
evidence to be admissible. Cryptographic hashing provides the
technical mechanism for proving that evidence integrity has been
maintained throughout the chain.

### Tools Used

| Tool | Purpose |
|------|---------|
| `ps aux` | Capture running process list from live system |
| `netstat -tulnp` / `ss` | Document active network connections |
| `who` / `last` | Identify logged-in and recently logged-in users |
| `lsof` | List open file handles on the live system |
| `md5sum` / `sha256sum` | Generate cryptographic hashes of evidence files |
| `dd` | Create bit-for-bit forensic copies of storage media |

### Lab Observations

The live response exercise targeted a compromised Docker container
as the triage subject. Evidence collection followed the order of
volatility: running processes were captured with `ps aux`, active
network connections documented with `ss -tulnp`, and open file
handles recorded with `lsof`. Each output was written to a
timestamped file, and SHA256 hashes were immediately computed and
recorded for each evidence file using `sha256sum`, establishing
the cryptographic baseline for chain of custody. The exercise
demonstrated that a compromised container running a suspicious
outbound connection would have provided no network evidence had
the container been stopped before live triage was performed.

### Connection to Defensive Practice

Live response capability is one of the most time-critical skills
in incident response because the window for volatile evidence
collection closes the moment a system loses power or a process
terminates. Organizations that lack trained live response capability
frequently destroy the most valuable forensic evidence during the
initial response — by rebooting a compromised server, terminating
suspicious processes, or shutting down affected containers before
memory capture. A responder who understands the order of volatility
and can execute live triage commands under pressure preserves the
evidence chain required for attribution, legal proceedings, and
post-incident remediation validation.

---

## Session 29: The Digital Autopsy
(Memory Forensics, Disk Imaging & File Recovery)

### Summary

Session 29 advanced from live system triage to post-mortem forensic
analysis, covering the acquisition and examination of disk images
and the recovery of deleted files using command-line forensic tools.
The session introduced The Sleuth Kit as a professional forensic
toolkit for raw disk analysis and demonstrated that data marked as
deleted by the operating system frequently remains recoverable until
the storage sectors are overwritten.

### Key Concepts

**Disk Imaging:** A forensic disk image is a bit-for-bit copy of
a storage device that captures every sector — including deleted
files, slack space, and unallocated regions — rather than just
the files visible to the operating system. Forensic analysis is
performed on the image rather than the original device to prevent
any risk of evidence modification. The `dd` utility creates raw
disk images suitable for forensic analysis (Carrier, 2005).

**File System Structure:** Understanding how a file system organizes
data is prerequisite to forensic recovery. In most Linux file
systems, deleting a file removes the directory entry pointing to
the file's data but does not immediately overwrite the data blocks
themselves. The inode — the metadata structure recording the file's
size, permissions, timestamps, and data block locations — may
remain accessible until it is reallocated, providing a recovery
path for recently deleted files.

**The Sleuth Kit:** The Sleuth Kit (TSK) is an open-source
collection of command-line forensic tools for analyzing disk images
and recovering evidence. Key tools include `fsstat` for file system
metadata, `fls` for listing files including deleted entries, `icat`
for extracting file content by inode number, and `mmls` for
examining partition table structure (Carrier, 2005).

**MFT Analysis:** On Windows NTFS file systems, the Master File
Table (MFT) contains a record for every file and directory on the
volume, including files that have been deleted. MFT records persist
after deletion and contain metadata including file name, timestamps,
size, and data location, providing a forensic record of file system
activity even after files are removed through normal operating
system operations.

**Deleted File Recovery:** File carving is the process of recovering
files from a disk image by identifying file signatures — known
byte sequences at the beginning and end of specific file types —
in unallocated space. This technique can recover files even when
the file system metadata has been overwritten, as long as the data
blocks themselves remain intact.

### Tools Used

| Tool | Purpose |
|------|---------|
| `dd` | Create bit-for-bit forensic disk image |
| `sha256sum` | Hash disk image to verify acquisition integrity |
| `mmls` | Examine partition table of disk image |
| `fsstat` | Display file system metadata and statistics |
| `fls` | List files and directories including deleted entries |
| `icat` | Extract file content by inode number |
| `file` | Identify file type from binary signature |

### Lab Observations

The session acquired a forensic disk image from a target container
using `dd` and immediately computed and recorded the SHA256 hash
of the image to establish acquisition integrity. The Sleuth Kit
analysis began with `mmls` to identify the partition layout, followed
by `fsstat` to examine file system metadata. The `fls` command
with the `-r` flag recursively listed all file system entries
including deleted files, which were identifiable by an asterisk
prefix in the output. Deleted malware artifacts were recovered
using `icat` with the inode numbers identified in the `fls` output,
and the `file` command confirmed the recovered file types. The
exercise demonstrated that attacker tools deleted after use remained
fully recoverable from the disk image.

### Connection to Defensive Practice

Disk forensics capability is essential for post-incident
investigation because attackers routinely attempt to cover their
tracks by deleting tools, logs, and artifacts. Understanding that
deletion does not equal destruction informs both defensive
investigation and organizational policy: forensic investigation
of compromised systems should always begin with imaging before any
remediation activity, and data retention policies should account
for the forensic value of disk images in incident investigations.
The ability to recover deleted malware samples from a compromised
system provides critical intelligence for understanding the specific
tools and techniques used in an attack.

---

## Session 30: The Central Nervous System
(SIEM Navigation & Log Correlation)

### Summary

Session 30 introduced Security Information and Event Management
systems as the central platform for log aggregation, correlation,
and attack timeline reconstruction at enterprise scale. The session
covered navigation and querying of the ELK Stack through the Kibana
interface, the construction of queries to identify specific attack
indicators across large log datasets, and the synthesis of disparate
log entries into a coherent attack timeline.

### Key Concepts

**Security Information and Event Management (SIEM):** A SIEM
platform collects, normalizes, and correlates log data from across
an organization's infrastructure — endpoints, network devices,
applications, and security controls — into a centralized repository
that enables detection, investigation, and response at a scale
impossible with manual log review. SIEM platforms provide real-time
alerting on rule-matched events and historical query capability
for forensic investigation (Chuvakin et al., 2013).

**ELK Stack:** The ELK Stack is an open-source SIEM platform
comprising three components: Elasticsearch, a distributed search
and analytics engine that stores and indexes log data; Logstash,
a data processing pipeline that ingests, transforms, and forwards
log data to Elasticsearch; and Kibana, a web-based visualization
and query interface that provides analysts with access to the
Elasticsearch data store. The ELK Stack is widely deployed as a
cost-effective alternative to commercial SIEM platforms in
organizations of all sizes.

**Log Correlation:** Log correlation is the process of identifying
relationships between events recorded in different log sources to
reconstruct the sequence of actions in an attack chain. A single
log entry rarely tells a complete story; the correlation of an
authentication failure in an auth log, followed by a successful
login from the same IP, followed by a process execution event on
the authenticated host, followed by an outbound connection to an
unknown IP, constitutes a correlated attack narrative that no
individual log source would reveal independently (Chuvakin et al.,
2013).

**Attack Timeline Reconstruction:** An attack timeline is a
chronologically ordered record of attacker actions derived from
correlated log evidence, documenting the sequence from initial
access through post-exploitation activities. A complete timeline
enables the organization to determine the extent of compromise,
identify all affected systems, and establish the point at which
remediation must begin to ensure complete eradication.

**Kibana Query Language (KQL):** KQL is the query language used
in Kibana to filter and search log data stored in Elasticsearch.
Queries can filter by field value, time range, and boolean
combinations, enabling analysts to isolate specific event types —
such as all authentication failures from a specific IP address
within a defined time window — from datasets containing millions
of records.

### Tools Used

| Tool | Purpose |
|------|---------|
| Kibana | Web interface for SIEM log query and visualization |
| KQL | Query language for filtering Elasticsearch log data |
| Elasticsearch | Backend log storage and indexing engine |
| Dashboard panels | Visualize event frequency and distribution |
| Timeline view | Reconstruct chronological attack sequence |

### Lab Observations

The session navigated the Kibana interface to query a pre-loaded
log dataset containing events from a simulated breach. KQL queries
filtered authentication logs for failure events, identified the
source IP responsible for a brute-force sequence, and traced the
subsequent successful authentication to a specific user account.
Cross-referencing the authentication timestamp against process
execution logs revealed a command execution event matching the
attacker's tooling, and network flow logs confirmed an outbound
connection to an external IP immediately following the execution
event. The correlated entries were assembled into a chronological
attack timeline documenting each stage from initial brute-force
through data exfiltration, demonstrating that events individually
invisible in any single log source became a clear attack narrative
when correlated across sources.

### Connection to Defensive Practice

SIEM proficiency is the core technical competency of SOC analyst
roles at every tier. The ability to construct targeted queries,
correlate events across log sources, and reconstruct attack
timelines from raw log data is what separates a security analyst
from a log reviewer. The attack timeline constructed in this session
reflects the output expected of a Tier 2 SOC analyst during an
active investigation — a structured, evidence-based narrative that
supports containment decisions, stakeholder communication, and
post-incident remediation planning. Organizations that cannot
reconstruct attack timelines from their log data lack the visibility
required to verify that eradication is complete and that the same
attack vector cannot be used again.

---

## TLAB 10: Operation Phantom Pursuit
(Full DFIR Investigation)

### Summary

The Week 10 Take-Home Lab integrated all three session skill sets
into a complete forensic investigation. Operation Phantom Pursuit
required SIEM-based log correlation to identify the initial attack
vector and lateral movement path, live triage of a compromised
container to capture volatile evidence and establish chain of
custody through cryptographic hashing, and disk image acquisition
and analysis using The Sleuth Kit to recover deleted attacker
tools from unallocated space. The completed investigation produced
a structured incident report documenting the full attack timeline,
evidence collected, and chain of custody records.

### Connection to Defensive Practice

Operation Phantom Pursuit reflected the complete DFIR workflow
executed during real-world breach investigations. The integration
of SIEM correlation, live triage, and disk forensics into a single
investigation mirrors the multi-source evidence collection required
to answer the three questions every organization asks after a
breach: what happened, how did they get in, and what did they
take. The ability to answer these questions with cryptographically
verified evidence rather than speculation is the defining
competency of a professional incident responder.

---

## References

Carrier, B. (2005). *File system forensic analysis*. Addison-Wesley.

Casey, E. (2011). *Digital evidence and computer crime: Forensic
    science, computers, and the internet* (3rd ed.). Academic Press.

Chuvakin, A., Schmidt, K., & Phillips, C. (2013). *Logging and log
    management: The authoritative guide to understanding the concepts
    surrounding logging and log management*. Syngress.

NIST. (2012). *Computer security incident handling guide* (SP
    800-61 Rev. 2).
    https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf
