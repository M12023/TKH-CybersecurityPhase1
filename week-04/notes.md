# Week 04 — Session Notes
**Operator:** Maurice Ratiff III
**Topic:** Infrastructure Hardening — Virtual Machines, Containers,
and Network Perimeter Defense
**Date:** May 28th 2026

---

## Summary

Week 4 extended the scripting and automation competencies developed
in prior weeks into the domain of infrastructure hardening. Across
three sessions, coursework examined the three critical layers of
secure enterprise infrastructure: the virtual machine, the container,
and the network perimeter. Session 10 introduced hypervisor
architecture and virtual network isolation. Session 11 contrasted
container-based OS virtualization with hardware virtualization and
established core Docker workflow competencies. Session 12 advanced
from single-container management to declarative multi-container
orchestration with air-gapped network segmentation. The cumulative
effect of the week was a transition from operating within secure
environments to designing and building them.

---

## Session 10: The Ghost in the Machine
(Virtualization & Hypervisors)

### Summary

Session 10 introduced the architectural foundations of virtualization,
distinguishing between Type 1 and Type 2 hypervisors and examining
how virtual networking modes control the exposure of virtual machines
to external networks. The session culminated in the deployment of a
forensic sandbox environment in which a simulated malware payload was
detonated under controlled, isolated conditions.

### Key Concepts

**Type 1 vs. Type 2 Hypervisors:** A hypervisor is software that
creates and manages virtual machines by abstracting physical hardware
resources. Type 1 hypervisors, also known as bare-metal hypervisors,
run directly on the host hardware without an underlying operating
system, providing greater performance and security isolation. Type 2
hypervisors run as applications on top of a host operating system,
making them more accessible for desktop and lab use but introducing
an additional layer between the VM and the hardware (Tanenbaum & Bos,
2014). VirtualBox is a Type 2 hypervisor widely used in security
training environments.

**Virtual Networking Modes:** VirtualBox supports multiple virtual
networking configurations that determine how a VM communicates with
external networks. Bridged mode connects the VM directly to the
physical network, assigning it a routable IP address visible to other
hosts. NAT mode allows outbound connectivity through the host's IP
address while blocking unsolicited inbound connections. Host-Only
mode creates an isolated network segment between the VM and the host,
with no path to external networks — making it the appropriate
configuration for malware analysis and forensic sandboxing (Oracle,
2023).

**Forensic Sandbox:** A forensic sandbox is an isolated environment
in which potentially malicious software can be executed and observed
without risk of lateral spread to production systems or external
networks. The effectiveness of a sandbox depends entirely on the
completeness of its network isolation; a sandbox misconfigured in
Bridged or NAT mode may allow a networked malware sample to
communicate with command-and-control infrastructure or propagate to
adjacent hosts.

**Detonation Testing:** Detonation testing refers to the deliberate
execution of a suspicious payload within a controlled environment to
observe its behavior. In this session's context, a provisioning
script seeded a simulated malware payload in a quarantine zone, and
connectivity testing via `ping` was used to confirm that the sandbox
isolation prevented external communication before and after
detonation.

### Tools Used

| Tool | Purpose |
|------|---------|
| VirtualBox Network Settings | Switch VM from Bridged to Host-Only mode |
| `ping` | Verify network isolation before and after configuration |
| Provisioning script | Seed simulated malware payload in quarantine zone |
| `ip addr` | Confirm VM network interface configuration |

### Lab Observations

The micro-lab demonstrated the practical difference between Bridged
and Host-Only networking by first confirming external connectivity
in Bridged mode via a successful `ping` to an external address, then
switching the adapter to Host-Only mode and confirming that the same
`ping` failed, verifying isolation. The deep dive provisioning script
seeded a simulated payload in a designated quarantine directory. Post-
detonation `ping` tests confirmed that no outbound network path
existed from within the sandboxed environment, and findings were
documented in `sandbox_report.txt` as a forensic record of the
isolation test.

### Connection to Defensive Practice

Virtualization is a foundational technology in both offensive and
defensive security. Malware analysts depend on correctly configured
sandbox environments to safely examine malicious samples without
exposing production infrastructure. A misconfigured sandbox — one
that retains external network access — can allow an analyzed sample
to phone home, download additional payloads, or propagate laterally.
Understanding hypervisor networking at a configuration level allows
a defender to verify sandbox integrity before detonation and audit
existing analysis environments for misconfiguration.

---

## Session 11: The Container Revolution
(Docker Fundamentals)

### Summary

Session 11 introduced container-based virtualization as a distinct
architectural model from hardware virtualization, and established
core Docker workflow competencies including image management, container
lifecycle operations, log auditing, and deployment scripting. The
session emphasized the security properties of container isolation
and the operational advantages of disposable, scriptable environments.

### Key Concepts

**Containers vs. Virtual Machines:** While a virtual machine emulates
complete hardware and runs a full guest operating system, a container
shares the host kernel and isolates only the process namespace,
filesystem, and network stack. This architectural difference makes
containers significantly more lightweight and faster to start than
VMs, but also means that a container escape vulnerability — one that
breaks out of the namespace isolation — grants access to the host
kernel directly (Merkel, 2014).

**The Docker Trinity:** Docker's core workflow revolves around three
interdependent concepts. A Dockerfile is a declarative text file
containing instructions for building a container image. An image is
an immutable, layered filesystem snapshot built from a Dockerfile.
A container is a running instance of an image, isolated from the
host through Linux namespaces and control groups (cgroups).
Understanding the relationship between these three components is
prerequisite to working with any containerized application.

**Linux Namespaces:** Namespaces are a kernel feature that provides
process isolation by giving each container its own view of system
resources including the process tree, network interfaces, filesystem
mounts, and user IDs. The `ps aux` command executed inside a
container reveals only the processes running within that container's
namespace, not those of the host or other containers, confirming
the isolation boundary (Kerrisk, 2010).

**Disposable Infrastructure:** The Docker workflow treats containers
as ephemeral rather than persistent. A container is created,
used, and destroyed — with no expectation of state persistence
between runs unless explicitly configured. This disposability model
has significant security implications: a compromised container can
be destroyed and replaced from a known-good image in seconds,
eliminating the need to remediate a potentially infected filesystem.

### Tools Used

| Tool | Purpose |
|------|---------|
| `docker pull` | Download a container image from a registry |
| `docker run -it` | Launch a container interactively |
| `docker ps` | List running containers |
| `docker logs` | Audit container output logs |
| `docker stop` / `docker rm` | Stop and destroy a container cleanly |
| `ps aux` | Verify process isolation within a container |

### Lab Observations

The micro-lab pulled the Alpine Linux image and launched it
interactively, confirming via `ps aux` that only the shell process
was visible within the container namespace — no host processes were
present. The deep dive deployed an Nginx web server container,
modified its default HTML content by writing to the container's
filesystem, confirmed the change via a `curl` request, audited the
access logs with `docker logs`, and then destroyed the container
completely with `docker stop` and `docker rm`. The entire sequence
was then encoded into `deploy_web.sh`, a reusable shell script that
reproduced the full deployment and destruction workflow with a single
command.

### Connection to Defensive Practice

Containers are the dominant deployment model in modern cloud and
enterprise infrastructure, making Docker fluency essential for any
security practitioner working in these environments. From a defensive
perspective, the disposability of containers is a significant
advantage: an incident response action that would take hours on a
traditional server — reimaging, reconfiguring, and validating a
clean state — takes seconds in a containerized environment. Log
auditing via `docker logs` provides a first-pass visibility tool
for identifying anomalous behavior within a running container before
a full forensic investigation is required.

---

## Session 12: The Conductor and the Fleet
(Docker Compose & Orchestration)

### Summary

Session 12 advanced from imperative single-container Docker commands
to declarative multi-container orchestration using Docker Compose
and YAML configuration files. The session introduced network
segmentation within a Compose stack, culminating in the deployment
of a WordPress and MySQL application in which the web tier and
database tier were isolated in separate bridge networks with no
shared external access for the database.

### Key Concepts

**Docker Compose:** Docker Compose is a tool for defining and running
multi-container applications through a declarative YAML configuration
file. Rather than issuing individual `docker run` commands for each
service, a `docker-compose.yml` file specifies all services,
networks, and volumes in a single document that can be launched with
`docker-compose up` and torn down with `docker-compose down`
(Docker, Inc., 2023).

**YAML Syntax:** YAML is a human-readable data serialization format
used extensively in infrastructure configuration. Docker Compose
files use YAML to define service names, images, environment
variables, port mappings, network memberships, and volume mounts.
Indentation is syntactically significant in YAML, making precise
formatting essential for valid configuration files.

**Bridge Networks:** Docker bridge networks are isolated Layer 2
network segments that allow containers attached to the same bridge
to communicate while blocking traffic from containers on other
bridges. By assigning services to different bridge networks, a
Compose stack can enforce network segmentation equivalent to VLAN
separation within a single host.

**Air-Gapped Database Networks:** Setting `internal: true` on a
Docker Compose network creates a bridge with no external routing —
containers on this network can communicate with each other but
cannot initiate or receive connections from outside the Docker
environment. This configuration mirrors the network isolation
applied to production database tiers, where direct external access
to the database represents a critical security risk (Docker,
Inc., 2023).

### Tools Used

| Tool | Purpose |
|------|---------|
| `docker-compose up -d` | Launch a multi-container stack in detached mode |
| `docker-compose down` | Stop and remove all stack containers and networks |
| `docker-compose ps` | Verify running services within a Compose stack |
| `docker network ls` | List active Docker networks |
| `docker network inspect` | Examine network membership and configuration |

### Lab Observations

The micro-lab exercise produced a minimal `docker-compose.yml` file
defining two services on a shared bridge network, confirmed both
containers launched successfully via `docker-compose ps`, and
demonstrated clean teardown with `docker-compose down`. The deep
dive deployed a WordPress and MySQL stack with two distinct networks:
a FrontEnd bridge connecting WordPress to external port 80, and a
BackEnd bridge configured with `internal: true` connecting WordPress
to MySQL with no external routing. Network inspection confirmed that
the MySQL container had no external network path while remaining
reachable from the WordPress container through the internal bridge,
successfully demonstrating database air-gapping within a Compose
stack.

### Connection to Defensive Practice

Multi-container orchestration with network segmentation directly
implements the principle of defense in depth at the infrastructure
layer. An attacker who compromises the web tier of a correctly
segmented Compose stack cannot reach the database tier directly —
they must first pivot through the web container, which operates
under its own isolation constraints. This architecture mirrors the
segmented deployment patterns used in production cloud environments
and provides a hands-on foundation for understanding and auditing
container security configurations in enterprise infrastructure.

---

## References

Docker, Inc. (2023). *Docker Compose overview*.
    https://docs.docker.com/compose/

Kerrisk, M. (2010). *The Linux programming interface: A Linux and
    UNIX system programming handbook*. No Starch Press.

Merkel, D. (2014). Docker: Lightweight Linux containers for
    consistent development and deployment. *Linux Journal, 2014*(239).
    https://dl.acm.org/doi/10.5555/2600239.2600241

Oracle. (2023). *Oracle VM VirtualBox user manual*.
    https://www.virtualbox.org/manual/

Tanenbaum, A. S., & Bos, H. (2014). *Modern operating systems*
    (4th ed.). Pearson.
