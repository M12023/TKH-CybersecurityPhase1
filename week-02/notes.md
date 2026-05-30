# Week 02 — Session Notes
**Topic:** Networking · Subnetting · Protocol Interrogation
**Date:** May 2026

---

## Summary

Week 2 extended foundational Linux competencies into the domain of
network engineering and protocol analysis. Across three sessions,
coursework examined the OSI model's physical through application
layers, with emphasis on interface recovery, subnet mathematics, and
application-layer service interrogation. The cumulative Take-Home Lab
required students to apply all three skill sets in sequence to respond
to a simulated server compromise, reinforcing the real-world
interdependence of networking knowledge in incident response contexts.

---

## Session 4: The Wire (L1–L3)

### Summary

Session 4 introduced Layer 1 through Layer 3 networking concepts with
an emphasis on practical interface diagnostics and route restoration.
The session simulated a common production scenario in which a network
interface is down and connectivity must be restored manually through
the command line without access to a GUI or network management tool.

### Key Concepts

**OSI Model Layers 1–3:** The OSI model defines a layered framework
for network communication. Layer 1 (Physical) governs the transmission
medium, Layer 2 (Data Link) manages hardware addressing via MAC
addresses, and Layer 3 (Network) handles logical addressing and routing
through IP (Tanenbaum & Bos, 2014). Proficiency at these layers is
prerequisite to diagnosing the majority of network connectivity
failures encountered in enterprise environments.

**Network Interfaces:** A network interface is the point of
interconnection between a host and a network. Each interface is
assigned a hardware MAC address at Layer 2 and one or more IP
addresses at Layer 3. The `ip link` command exposes interface state
and hardware addressing, while `ip addr` reveals logical address
assignments.

**Default Gateway:** The default gateway is the Layer 3 device
responsible for forwarding traffic destined for addresses outside the
local subnet. When the default gateway route is absent from a host's
routing table, the host loses the ability to reach external networks
regardless of whether its interface is active (Tanenbaum & Bos, 2014).

**Route Restoration:** Manual route restoration is performed using the
`ip route add` command. In scenarios where the default gateway has
been removed or was never configured, this command re-establishes the
path to external networks without requiring a system restart.

### Tools Used

| Tool | Purpose |
|------|---------|
| `ip link` | Identify physical interfaces and their up/down state |
| `ip addr` | View IP address assignments per interface |
| `ip route` | Inspect and modify the host routing table |
| `ip route add` | Manually restore a default gateway route |
| `ping` | Verify connectivity after route restoration |

### Lab Observations

The Operation Broken Link exercise presented a terminal with a downed
network interface and no default gateway route. The diagnostic process
began with `ip link show` to identify the inactive interface, followed
by `ip link set [interface] up` to bring it online. The default
gateway was then restored using `ip route add default via [gateway IP]`,
after which external connectivity was confirmed via `ping`. The
exercise demonstrated that network connectivity failures are frequently
configuration issues rather than hardware failures, and that manual
restoration is achievable without specialized tooling.

### Connection to Defensive Practice

Network interface and routing table manipulation are common techniques
in both legitimate administration and adversarial activity. An attacker
who gains access to a host may modify routing tables to redirect
traffic or isolate the host from monitoring infrastructure. A defender
who understands routing at this level can detect anomalous route
entries during incident response and restore expected network behavior
rapidly, minimizing the dwell time of a network-based attack.

---

## Session 5: The Subnetting Crucible

### Summary

Session 5 examined CIDR notation and subnet mathematics with emphasis
on understanding how subnet masks define network boundaries and how
misconfigured masks produce host isolation. The Binary Surgery exercise
used Python to visualize the bitwise relationship between IP addresses
and subnet masks, grounding the mathematical concepts in observable
output.

### Key Concepts

**CIDR Notation:** Classless Inter-Domain Routing (CIDR) notation
expresses both an IP address and its associated subnet mask in a
single value, such as `192.168.1.0/24`. The prefix length following
the slash indicates the number of bits reserved for the network
portion of the address, with the remaining bits available for host
addressing (IETF, 1993).

**Subnet Mask:** A subnet mask is a 32-bit value that divides an IP
address into its network and host components. When applied via a
bitwise AND operation against an IP address, the mask reveals the
network address to which the host belongs. Hosts on different subnets
cannot communicate directly without a Layer 3 routing device.

**The Magic Number:** The "magic number" method provides a practical
shortcut for subnet calculation. Subtracting the relevant octet of
the subnet mask from 256 yields the block size of the subnet,
allowing rapid identification of network boundaries, broadcast
addresses, and valid host ranges without binary conversion.

**Host Isolation via CIDR Mismatch:** When a host is configured with
an incorrect subnet mask, it may mathematically place itself on a
different network than its peers, preventing direct communication
even when physical connectivity is intact. This type of misconfiguration
is a common source of intermittent or total connectivity loss in
enterprise environments.

### Tools Used

| Tool | Purpose |
|------|---------|
| `ip addr` | View current subnet mask configuration |
| `ip addr add` | Assign a corrected IP address and prefix length |
| `ipcalc` | Calculate network boundaries from CIDR notation |
| Python (`bin()`) | Convert IP octets to binary for visual inspection |

### Lab Observations

The Binary Surgery mini-lab used Python's built-in `bin()` function
to convert each octet of an IP address and subnet mask into an 8-bit
binary string, making the bitwise relationship between address and
mask directly visible. The Operation Grid Lock exercise presented a
host configured with an incorrect `/25` mask instead of the correct
`/24`, placing it on a mathematically separate subnet from its
gateway. Correcting the mask with `ip addr add` restored communication
immediately, confirming that the underlying physical connectivity had
been intact throughout.

### Connection to Defensive Practice

Subnet misconfiguration is a vector for both accidental outages and
deliberate network segmentation bypass. In a hardened environment,
subnet boundaries serve as a primary control for limiting lateral
movement — a host on the application tier subnet should not be able
to reach the database tier directly. A defender who understands CIDR
mathematics can audit subnet configurations to verify that segmentation
controls are functioning as designed, and can detect cases where
hosts have been reconfigured to bridge subnet boundaries.

---

## Session 6: Protocol Interrogation

### Summary

Session 6 examined the application layer with focus on DNS resolution,
port enumeration, and the detection of services operating on
non-standard ports. The session introduced the concept of DNS
poisoning through manipulation of the `/etc/hosts` file and required
students to remediate a compromised name resolution configuration
while locating a concealed web service.

### Key Concepts

**Application Layer Protocols:** The application layer of the OSI
model encompasses the protocols through which user-facing services
communicate, including HTTP, DNS, SSH, and SMTP. Understanding which
ports these services occupy by default — and how to detect deviations
from those defaults — is fundamental to both network administration
and security monitoring (Tanenbaum & Bos, 2014).

**DNS Resolution:** The Domain Name System translates human-readable
hostnames into IP addresses. DNS resolution follows a hierarchical
process beginning with the local `/etc/hosts` file before querying
configured DNS servers. Entries in `/etc/hosts` take precedence over
external DNS, making this file a high-value target for attackers
seeking to redirect traffic on a compromised host (Albitz & Liu, 2006).

**`/etc/hosts` Poisoning:** An attacker with write access to
`/etc/hosts` can redirect any hostname to an arbitrary IP address,
effectively performing a local man-in-the-middle attack without
touching network infrastructure. Detection requires direct inspection
of the file's contents and comparison against expected hostname
mappings.

**Non-Standard Port Services:** Services are not required to operate
on their default ports. A web server configured to listen on port 8080
or 4444 instead of port 80 may evade cursory port checks. Comprehensive
service discovery requires scanning the full port range rather than
relying on default port assumptions.

### Tools Used

| Tool | Purpose |
|------|---------|
| `ss -tulnp` | Enumerate open ports and associated processes |
| `dig` | Query DNS servers and inspect resolution responses |
| `curl` | Send HTTP requests to services on non-standard ports |
| `cat /etc/hosts` | Inspect local hostname override file |
| `nano /etc/hosts` | Remediate poisoned hostname entries |

### Lab Observations

The Operation Hidden Door exercise began with an `/etc/hosts` file
containing a fabricated entry that redirected a legitimate hostname
to an attacker-controlled IP address. After removing the malicious
entry, `dig` was used to confirm that DNS resolution returned to the
correct address. A full port scan using `ss` revealed a web service
operating on a non-standard port, which was then interrogated using
`curl` to capture its response headers and confirm the service
identity. The exercise demonstrated that effective service discovery
requires both DNS validation and comprehensive port enumeration.

### Connection to Defensive Practice

Protocol interrogation skills are directly applicable to threat
hunting and incident response. An analyst investigating a compromised
host should routinely inspect `/etc/hosts` for unauthorized entries,
enumerate all listening services regardless of port, and validate DNS
resolution behavior against expected baselines. These checks can
reveal persistence mechanisms, data exfiltration channels, and
command-and-control infrastructure that would not be visible through
perimeter monitoring alone.

---

## TLAB: Operation Silent Ghost

### Summary

The Week 2 Take-Home Lab simulated a SOC analyst response to a
confirmed server compromise. The mission required sequential
application of all three session skill sets: restoring a downed
network interface (Session 4), correcting a subnet mask misconfiguration
to reach an internal database (Session 5), and identifying a hidden
exfiltration service operating on a non-standard port to capture its
headers (Session 6). The artifact produced was `tlab_report.txt`.

### Connection to Defensive Practice

The cumulative structure of Operation Silent Ghost reflected the
reality of incident response, in which analysts rarely encounter
isolated problems. A real compromise frequently involves multiple
simultaneous indicators across the network, configuration, and
application layers. The ability to diagnose and remediate issues at
each layer in sequence — without losing state or context — is a
core competency for SOC analysts and network defenders.

---

## References

Albitz, P., & Liu, C. (2006). *DNS and BIND* (5th ed.). O'Reilly
    Media.

IETF. (1993). *Classless inter-domain routing (CIDR): An address
    assignment and aggregation strategy* (RFC 1519).
    https://www.rfc-editor.org/rfc/rfc1519

Tanenbaum, A. S., & Bos, H. (2014). *Modern operating systems*
    (4th ed.). Pearson.
