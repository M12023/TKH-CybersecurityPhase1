# Week 05 — Session Notes
**Operator:** Maurice Ratiff III
**Topic:** Identity, Access Management (IAM), and Windows Enterprise
Infrastructure
**Date:** May 2026

---

## Summary

Week 5 extended infrastructure competencies into the domain of
centralized identity and access management, establishing the principle
that in modern enterprise environments, identity functions as the
primary security perimeter. Across three sessions, coursework covered
the promotion of a Windows Server to a Domain Controller, the
deployment of Group Policy Objects to enforce system-wide security
controls, and the integration of Linux endpoints into a Windows Active
Directory domain. The cumulative Take-Home Lab required students to
build a fully governed corporate domain from scratch, synthesizing
all three session skill sets into a complete enterprise identity
architecture.

---

## Session 13: The Corporate Brain
(Windows Server & Active Directory)

### Summary

Session 13 introduced Active Directory as the centralized identity
and access management system of Windows enterprise environments.
The session covered the promotion of a standalone Windows Server 2022
instance to a Domain Controller, the architectural hierarchy of
Active Directory forests, domains, and organizational units, and the
automation of user account provisioning at scale using PowerShell.

### Key Concepts

**Active Directory:** Active Directory (AD) is a directory service
developed by Microsoft that provides centralized authentication and
authorization for Windows domain environments. AD stores information
about users, computers, and other network objects, and enforces
access controls through a combination of group memberships, policies,
and the Kerberos authentication protocol (Microsoft, 2023a).

**Domain Controller:** A Domain Controller (DC) is a Windows Server
instance that hosts the Active Directory database and handles
authentication requests for all domain-joined machines. Promoting a
server to a Domain Controller installs the AD Domain Services role
and establishes the server as the authoritative identity source for
the domain. All subsequent user authentication and policy enforcement
flows through the Domain Controller.

**Forest, Domain, and Organizational Unit Hierarchy:** Active
Directory organizes objects in a three-tier hierarchy. A forest is
the outermost boundary of an AD environment, containing one or more
domains that share a common schema. A domain is the primary
administrative unit, containing users, computers, and groups. An
Organizational Unit (OU) is a container within a domain used to
organize objects and apply Group Policy at a granular level
(Microsoft, 2023a).

**Kerberos Authentication:** Active Directory uses the Kerberos
protocol for authentication, ensuring that passwords are never
transmitted over the network. Instead, the Domain Controller issues
time-limited cryptographic tickets that clients present to services
as proof of identity. This ticket-based model significantly reduces
the attack surface associated with credential interception compared
to older challenge-response protocols (Neuman & Ts'o, 1994).

**PowerShell Automation:** PowerShell is the primary scripting
environment for Windows administration. The `New-ADUser` cmdlet
creates user accounts in Active Directory programmatically, enabling
administrators to provision hundreds of accounts in the time it would
take to create a single account through the graphical interface.
Encapsulating provisioning logic in a `for` loop allows the same
script to scale from five users to five thousand without modification.

### Tools Used

| Tool | Purpose |
|------|---------|
| Server Manager | Install AD Domain Services role |
| `Install-ADDSForest` | Promote server to Domain Controller |
| Active Directory Users and Computers | Visual AD object management |
| PowerShell ISE | Write and execute provisioning scripts |
| `New-ADUser` | Create domain user accounts programmatically |
| `New-ADOrganizationalUnit` | Create OU containers within the domain |

### Lab Observations

The Forest Begins micro-lab promoted a standalone Windows Server 2022
instance to a Domain Controller by installing the AD Domain Services
role through Server Manager and running the domain promotion wizard
to establish a new forest named `titan.local`. Following promotion
and reboot, login as `TITAN\Administrator` confirmed successful domain
establishment. The Automated Onboarding deep dive created an
Engineering OU using PowerShell and then executed a `for` loop
using `New-ADUser` to provision five engineering accounts into the
OU simultaneously, demonstrating that identity management at
enterprise scale requires automation rather than manual account
creation. The artifact produced was `onboard_engineers.ps1`.

### Connection to Defensive Practice

Active Directory is the identity backbone of the majority of
enterprise Windows environments, making it one of the highest-value
targets for attackers operating inside a corporate network. Attacks
such as Pass-the-Hash, Kerberoasting, and DCSync all target Active
Directory specifically because compromising the Domain Controller
yields access to every domain-joined resource simultaneously (MITRE,
2023). A defender who understands how AD is architected — forests,
domains, OUs, and the Kerberos ticket system — can identify
anomalous authentication patterns, detect unauthorized privilege
escalation, and audit account provisioning for indicators of
compromise.

---

## Session 14: The Invisible Hand
(Group Policy Mastery)

### Summary

Session 14 introduced Group Policy Objects as the primary mechanism
for enforcing security configuration across Active Directory domain
environments at scale. The session covered GPO creation and linking,
the LSDOU policy inheritance model, forced policy application, and
compliance auditing — culminating in a written audit report
documenting enforcement logic and verification procedures.

### Key Concepts

**Group Policy Objects:** A Group Policy Object (GPO) is a collection
of configuration settings that can be applied to users and computers
within an Active Directory domain. GPOs can enforce security
settings, software restrictions, desktop configurations, and audit
policies across all machines within their scope simultaneously,
making them the primary tool for endpoint hardening at enterprise
scale (Microsoft, 2023b).

**LSDOU Inheritance Order:** When multiple GPOs apply to the same
object, Active Directory resolves conflicts using the LSDOU
inheritance order: Local policy is applied first, followed by Site
policy, then Domain policy, then Organizational Unit policy. Each
subsequent layer can override the previous, with OU-level policy
taking final precedence. Understanding LSDOU is essential for
predicting which setting will be in effect on any given endpoint
when policies conflict (Microsoft, 2023b).

**GPO Linking:** Creating a GPO does not apply it until the GPO is
linked to a specific scope — a site, domain, or OU. Linking a GPO
to an OU restricts its application to the objects within that OU,
enabling granular policy enforcement. A GPO linked to the Engineering
OU applies only to users and computers within that OU, leaving other
OUs unaffected.

**`gpupdate /force`:** Group Policy is applied at startup and at
regular background refresh intervals by default. The `gpupdate
/force` command bypasses the refresh interval and immediately
applies all applicable GPOs to the current machine, enabling
administrators to verify policy enforcement without waiting for the
next scheduled refresh cycle.

**`gpresult`:** The `gpresult` command generates a Group Policy
Results report for a specified user and computer, documenting which
GPOs were applied, which were filtered, and the effective setting
for each configured policy area. This output serves as the primary
compliance verification artifact for GPO auditing.

### Tools Used

| Tool | Purpose |
|------|---------|
| Group Policy Management Console | Create, configure, and link GPOs |
| `gpupdate /force` | Force immediate policy application |
| `gpresult /r` | Generate Group Policy Results report |
| `gpresult /h` | Export results as an HTML report |

### Lab Observations

The Control Panel Lock micro-lab created a GPO named
`Lockdown_ControlPanel`, configured it to prohibit access to the
Control Panel and Settings for all users, and linked it to the
Engineering OU. Logging in as a domain user in the Engineering OU
confirmed that Control Panel access was blocked, verifying successful
GPO enforcement. The Audit and Enforce deep dive ran `gpupdate
/force` to immediately apply all domain policies, then used
`gpresult /r` to generate a compliance report documenting which
GPOs were applied and their effective settings. Findings were
documented in `gpo_audit.txt`, including an explanation of the
LSDOU inheritance order and its implications for policy conflict
resolution.

### Connection to Defensive Practice

Group Policy is one of the most powerful tools available to a
Windows system administrator for enforcing security baselines across
an enterprise at scale. CIS Benchmarks and DISA STIGs — the two
primary standards frameworks for Windows endpoint hardening — are
largely implemented through GPO configurations (CIS, 2023). A
defender who understands GPO creation, linking, and inheritance can
translate security benchmark requirements directly into enforceable
policy, audit compliance across the domain without visiting
individual machines, and detect unauthorized GPO modifications that
may indicate an attacker attempting to weaken domain-wide security
controls.

---

## Session 15: Bridging the Kingdoms
(Linux Integration)

### Summary

Session 15 unified the Linux and Windows environments developed
across the course by joining an Ubuntu virtual machine to the
`titan.local` Active Directory domain. The session configured the
System Security Services Daemon (SSSD) for cross-platform
authentication and established a sudoers bridge granting Windows
Domain Administrators root-level privileges on the Linux endpoint,
demonstrating a single identity perimeter spanning two operating
systems.

### Key Concepts

**realmd:** The `realmd` utility provides a simplified interface for
discovering and joining Active Directory domains from Linux. The
`realm discover` command queries DNS for domain information, and
`realm join` performs the domain join operation, configuring the
necessary Kerberos and SSSD settings automatically (freedesktop.org,
2023).

**SSSD:** The System Security Services Daemon (SSSD) is a Linux
service that provides access to remote identity and authentication
providers, including Active Directory. Once configured, SSSD enables
Linux endpoints to authenticate users against the AD domain,
caching credentials for offline use and mapping Windows domain
accounts to local Linux user attributes.

**Cross-Platform Authentication:** Joining a Linux machine to an
Active Directory domain enables Windows domain users to authenticate
to the Linux endpoint using their domain credentials. This
eliminates the need to maintain separate local user accounts on each
Linux machine, centralizing identity management under the Active
Directory infrastructure already governing the Windows environment.

**Sudoers Bridge:** The `visudo` command opens the sudoers
configuration file, which governs which users and groups may execute
commands with elevated privileges on a Linux system. Configuring
a sudoers entry for the Windows Domain Admins group grants members
of that group root-level `sudo` access on the Linux endpoint,
completing the unified identity architecture.

### Tools Used

| Tool | Purpose |
|------|---------|
| `realm discover` | Query DNS for Active Directory domain information |
| `realm join` | Join the Linux machine to the AD domain |
| `id` | Verify domain user resolution on the Linux endpoint |
| `sssd.conf` | Configure SSSD for short name authentication |
| `visudo` | Configure sudoers bridge for Domain Admins group |
| `sudo whoami` | Verify root privilege for domain-authenticated user |

### Lab Observations

The Joining the Domain micro-lab installed the `realmd`, `sssd`,
and Kerberos prerequisite packages, ran `realm discover titan.local`
to confirm domain visibility, and executed `realm join titan.local`
with Domain Administrator credentials to complete the join. The
command `id Administrator@titan.local` returned a valid UID and
group membership, confirming that the Linux endpoint was resolving
domain identities correctly. The Unified Admin deep dive configured
SSSD short name resolution to allow login without the full domain
suffix, then added a sudoers entry granting the Domain Admins group
unrestricted `sudo` access. The final proof — executing `sudo
whoami` while authenticated as a Windows domain user and receiving
the response `root` — was captured as `unified_identity.png`,
demonstrating successful cross-platform identity unification.

### Connection to Defensive Practice

Hybrid identity environments — in which Linux and Windows endpoints
share a common authentication infrastructure — are standard in
enterprise organizations that run mixed operating system fleets.
Centralized identity management reduces the attack surface associated
with locally managed accounts: there are no stale local credentials
to exploit, no inconsistent password policies to bypass, and no
separately managed privilege assignments to audit. A defender who
can architect and audit cross-platform Active Directory integration
is equipped to enforce consistent identity governance across the
full scope of an enterprise environment.

---

## TLAB: Operation Sovereign Domain

### Summary

The Week 5 Take-Home Lab simulated the role of a Domain
Administrator tasked with building a secure corporate domain
environment from scratch. The mission required sequential application
of all three session skill sets: promoting a Windows Server to a
Domain Controller and provisioning domain users with PowerShell
(Session 13), deploying and verifying GPO-based access controls
across the domain (Session 14), and authenticating a Linux endpoint
against the Windows domain to demonstrate unified identity management
(Session 15). The completed environment represented a functional,
policy-governed enterprise identity architecture built entirely
within the lab infrastructure.

### Connection to Defensive Practice

Operation Sovereign Domain demonstrated that enterprise security is
not solely a technical discipline but an architectural one. The
decisions made during domain design — OU structure, GPO inheritance
scope, sudoers bridge configuration — have direct consequences for
the security posture of every machine and user account in the
environment. A defender who has built a domain from scratch
understands the attack surface of each architectural decision and
is better equipped to identify misconfigurations, detect anomalous
activity, and respond to identity-based attacks such as
Kerberoasting, Pass-the-Hash, and GPO tampering.

---

## References

CIS. (2023). *CIS Microsoft Windows Server 2022 benchmark*.
    https://www.cisecurity.org/benchmark/microsoft_windows_server

freedesktop.org. (2023). *realmd — Discover and join identity
    domains*. https://www.freedesktop.org/software/realmd/

Microsoft. (2023a). *Active Directory Domain Services overview*.
    https://learn.microsoft.com/en-us/windows-server/identity/
    ad-ds/get-started/virtual-dc/active-directory-domain-services-overview

Microsoft. (2023b). *Group Policy overview*.
    https://learn.microsoft.com/en-us/previous-versions/windows/
    it-pro/windows-server-2012-r2-and-2012/hh831791(v=ws.11)

MITRE. (2023). *MITRE ATT&CK: Design and philosophy*.
    https://attack.mitre.org/docs/ATTACK_Design_and_Philosophy_March_2020.pdf

Neuman, C., & Ts'o, T. (1994). Kerberos: An authentication service
    for computer networks. *IEEE Communications Magazine, 32*(9),
    33–38. https://doi.org/10.1109/35.312841
