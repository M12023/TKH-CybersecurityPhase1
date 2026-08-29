
## 3. Tools & Technologies
- **Access Control:** SSH (key-based authentication, root/password login disabled)
- **Network Security:** UFW (Uncomplicated Firewall) — default-deny policy, explicit port allow-listing
- **Automation & Monitoring:** Python (custom watchdog/audit script)
- **Containerization:** Docker, Docker networking (isolated frontend/backend bridge networks)
- **Application Stack:** nginx, MySQL 8.0

## 4. Results & Outcomes
- Eliminated password- and root-based SSH brute-force vectors by enforcing key-only authentication and disabling root login.
- Reduced the network attack surface to exactly two allowed inbound ports (22, 8082) under a default-deny firewall policy, verified via `ufw status verbose`.
- Deployed a working automated watchdog that logs infrastructure connectivity status to persistent log files, giving lightweight visibility into core infrastructure uptime.
- Achieved full network isolation between the application frontend and the database — confirmed the backend container exposes zero ports to the host, meaning the database is unreachable even if the frontend is compromised.
- Verified a healthy, running stack via `docker ps`, confirming both containers up and correctly bound to their intended networks and ports.

## 5. Project Evidence
- Full Security Architecture Document (firewall configuration output, container stack status, script logic): *(paste a link here if you have this written up as a separate file/doc in the repo, or a screenshot of the terminal output — otherwise this section can be left as the write-up in this README)*

## 6. Individual Contribution
This was solo work. I independently designed and implemented all three layers of this architecture — SSH and firewall hardening, the Python monitoring watchdog, and the isolated Docker network topology — and can speak to every configuration decision in an interview.
