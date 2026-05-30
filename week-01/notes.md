# Week 01 — Session Notes
**Topic:** Terminal Supremacy — The Command Line as a Secure Navigation Tool
**Date:** May 29th 2026

---

## Summary

Week 1 established the foundational command-line competencies required
for professional work in headless Linux environments. Across three
sessions, coursework covered the Filesystem Hierarchy Standard (FHS),
Linux file permission architecture, and text stream manipulation using
core utilities. These skills collectively form the operational baseline
for all subsequent cybersecurity work, as the ability to navigate,
interpret, and modify a Linux system efficiently is prerequisite to
both offensive and defensive practice.

---

## Session 1: The Shell Awakening

### Summary

Session 1 introduced the Linux command-line interface as the primary
tool for system navigation and administration. Emphasis was placed on
command anatomy, the Filesystem Hierarchy Standard, and the ability to
operate effectively in environments without a graphical interface.

### Key Concepts

**Command Anatomy:** Every Linux command follows a consistent structure
of `command [options] [arguments]`. Understanding this structure allows
a practitioner to read unfamiliar commands confidently and construct
precise instructions without relying on memorization alone.

**Filesystem Hierarchy Standard (FHS):** The FHS defines the directory
structure and content of Linux distributions. Key directories include
`/etc` for system configuration files, `/var` for variable data such
as logs, `/home` for user directories, and `/proc` for runtime kernel
and process information (Linux Foundation, 2015).

**Headless Navigation:** Operating without a graphical interface requires
fluency with commands such as `ls`, `cd`, `find`, `cat`, `less`, and
`pwd`. In production server and forensic environments, a GUI is rarely
available, making terminal fluency a non-negotiable professional skill.

### Tools Used

| Tool | Purpose |
|------|---------|
| `ls -la` | List directory contents including hidden files and permissions |
| `find` | Locate files by name, type, or attribute across the filesystem |
| `cat` / `less` | Read file contents directly in the terminal |
| `pwd` | Confirm current working directory during navigation |
| `tree` | Visualize multi-directory web application structures |

### Lab Observations

The Scavenger Hunt exercise required locating configuration files
distributed across multiple FHS directories, including `/etc`, `/var`,
and `/opt`. The exercise reinforced that knowing where Linux stores
specific file types by convention is as important as knowing the
commands to retrieve them. Files such as `/etc/passwd` and web
application configuration files were located and read without a GUI,
simulating a realistic post-compromise reconnaissance scenario.

### Connection to Defensive Practice

Configuration files contain some of the most sensitive data on a Linux
system, including service credentials, database connection strings, and
network parameters. A defender must know where these files live and
which processes should legitimately access them. Baseline familiarity
with the FHS allows a security analyst to quickly identify files that
are out of place or have been modified unexpectedly during incident
response.

---

## Session 2: The Permissions Gauntlet

### Summary

Session 2 examined the Linux file permission model in depth, covering
both octal and symbolic notation, special permission bits, and the
security implications of misconfigured access controls on sensitive
system files.

### Key Concepts

**Read-Write-Execute (RWX) Matrix:** Linux file permissions are applied
across three principal categories — owner, group, and others — with
read (r/4), write (w/2), and execute (x/1) permissions available for
each. The combined octal value of a file's permissions determines who
can interact with it and in what capacity.

**Octal vs. Symbolic Notation:** Permissions can be expressed
symbolically (e.g., `chmod u+x file`) or in octal (e.g., `chmod 755
file`). Octal notation is more common in scripts and documentation,
while symbolic notation is more readable during interactive sessions.

**Special Permission Bits:** The setuid (SUID), setgid (SGID), and
sticky bit extend the standard RWX model. SUID causes an executable
to run with the file owner's privileges rather than the invoking
user's, which represents a significant security risk if applied to
the wrong binary (Kerrisk, 2010).

**`/etc/shadow`:** This file stores hashed user passwords and is
readable only by root by design. Incorrect permissions on this file —
such as world-readable access — represent a critical vulnerability,
as it exposes password hashes to any local user.

### Tools Used

| Tool | Purpose |
|------|---------|
| `chmod` | Modify file and directory permissions |
| `chown` | Change file ownership |
| `ls -l` | Inspect current permission assignments |
| `stat` | View detailed file metadata including permissions and timestamps |
| `umask` | View and set default permission masks for new files |

### Lab Observations

The group-writable file creation exercise required precise use of
`chmod 664` and `chown :groupname` to produce files accessible to a
specific group without exposing them to all system users. The
`/etc/shadow` diagnostic exercise demonstrated how a single
misconfigured `chmod` command on a sensitive file could open a
critical attack vector, reinforcing that permission errors are not
abstract — they have direct, exploitable consequences.

### Connection to Defensive Practice

Misconfigured file permissions are among the most common findings in
Linux security audits. Tools such as Lynis and manual inspection of
SUID binaries via `find / -perm -4000` are standard components of
system hardening checklists. A defender who understands the permission
model deeply can identify privilege escalation vectors before an
attacker does, and can write monitoring rules that alert on unexpected
permission changes to sensitive files.

---

## Session 3: Stream Editing & Piping

### Summary

Session 3 introduced standard I/O streams, redirection, and the core
Unix text processing utilities. The session culminated in a pipeline-
based analysis of a 10,000-line Apache web server log to identify the
highest-volume source IP addresses.

### Key Concepts

**Standard Streams:** Linux defines three standard I/O streams —
stdin (0), stdout (1), and stderr (2). Redirection operators such as
`>`, `>>`, `2>`, and `2>&1` allow these streams to be routed between
commands and files, enabling powerful data transformation pipelines
without writing intermediate output to disk.

**`grep`:** A pattern-matching utility that filters lines from a text
stream based on a regular expression. In security contexts, grep is
frequently used to search logs for specific IP addresses, error codes,
or indicators of compromise.

**`sed`:** A stream editor that performs text substitution and
transformation on input line by line. Common uses include stripping
unwanted characters from logs and performing in-place configuration
file edits.

**`awk`:** A text-processing language that operates on structured
fields within each line of input. It is particularly effective for
extracting specific columns from log files, such as isolating IP
addresses from Apache access logs (Robbins, 2005).

### Tools Used

| Tool | Purpose |
|------|---------|
| `grep` | Filter log lines matching a specific pattern |
| `sed` | Transform and clean text streams |
| `awk` | Extract and aggregate structured fields from logs |
| `sort` / `uniq -c` | Sort and count unique values in a stream |
| `wc -l` | Count lines in a file or stream |
| `>` / `>>` / `2>` | Redirect stdout and stderr to files |

### Lab Observations

The Apache log analysis exercise processed a 10,000-line access log
to identify the top contributing IP addresses by request volume. The
pipeline constructed during the exercise was as follows:

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10
```

This single pipeline replaced what would otherwise require a custom
script, demonstrating the composability of Unix tools. The error log
stripping exercise used `grep -v` and stderr redirection to separate
error-level entries from informational output into dedicated files for
downstream analysis.

### Connection to Defensive Practice

Log analysis is one of the most time-critical skills in a SOC
environment. During an incident, an analyst may need to rapidly search
millions of log lines for a specific IP address, user agent, or
request pattern. Proficiency with grep, awk, and stream redirection
allows this work to be done directly on a server without transferring
large files or waiting for a SIEM query to complete. These tools also
serve as the building blocks of custom detection scripts and automated
triage pipelines.

---

## References

Kerrisk, M. (2010). *The Linux programming interface: A Linux and
    UNIX system programming handbook*. No Starch Press.

Linux Foundation. (2015). *Filesystem hierarchy standard (version
    3.0)*. https://refspecs.linuxfoundation.org/FHS_3.0/fhs-3.0.pdf

Robbins, A. (2005). *sed & awk* (2nd ed.). O'Reilly Media.
