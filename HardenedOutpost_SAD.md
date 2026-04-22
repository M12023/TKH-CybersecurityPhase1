# TITAN SMALL BUSINESS SERVICES: SECURITY ARCHITECTURE DOCUMENT (SAD)
**Operator:** Maurice Ratiff III
**Date:** 04/21/2026

 Perimeter Hardening (UFW & SSH)
* **SSH Status:**
 Root login and Password Authentication have been disabled. Access is strictly limited to Public Key Authentication. The ssh.socket has been reloaded to enforce these policies. 
Password authentication set to no and PermitRootLogin set to no to lock down the syste.
* **Firewall Logic:**
 Enabled the firewall using sudo ufw enable and sudo ufw default deny incoming and sudo ufw default allow outgoing.
With the ports being open 22 and 8080(Had to reassign to 8082 in order to prevent any network collisions due to having connection already in use errors. 
sudo ufw status verbose
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), deny (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere                  
8082/tcp                   ALLOW IN    Anywhere                  
22/tcp (v6)                ALLOW IN    Anywhere (v6)             
8082/tcp (v6)              ALLOW IN    Anywhere (v6)             
 The Automated Auditor (Python)
* **Script Logic:** 
import os
dc_ip = "192.168.1.161"
log_path = "/var/log/dc_audit.log"

response = os.system(f"ping -c 4 {dc_ip} > /dev/null 2>&1")
status = "DC is UP" if response == 0 else "DC is DOWN"

with open(log_path, "a") as f:

    f.write(status + "\n")
* **Telemetry Path:** /var/log/dc_audit.log (and /var/log/sys_audit.log

 Containerized App (Docker)
* **Network Isolation:** 
 The architecture utilizes a dual-bridge network strategy.
 The wiki service is connected to the frontend network to serve traffic on port 8082, while the db service is placed exclusively on the backend network.
 This "air-gap" ensures the database is unreachable from the host machine or external network.

* **Stack Health:**
 78aa3fd46ff8   nginx       "/docker-entrypoint.…"   52 seconds ago   Up 50 seconds   0.0.0.0:8082->80/tcp, [::]:8082->80/tcp   titan_frontend
titan_backend       mysql:8.0   Up 10 minutes  3306/tcp, 33060/tcp
 Executive Summary


 Perimeter Hardening (UFW & SSH)
SSH Status: Root login and Password Authentication have been disabled. Access is strictly limited to Public Key Authentication. The ssh.socket has been reloaded to enforce these policies.
sudo systemctl restart ssh.
Firewall Logic: UFW is configured to Deny all incoming traffic by default. Explicit Allow rules are active for Port 22/tcp (Management) and Port 8082/tcp (Application Frontend).

 The Automated Auditor (Python)
Script Logic:

Python
import os
dc_ip = "192.168.1.161"
log_path = "/var/log/dc_audit.log"

response = os.system(f"ping -c 4 {dc_ip} > /dev/null 2>&1")
status = "DC is UP" if response == 0 else "DC is DOWN"

with open(log_path, "a") as f:
    f.write(status + "\n")
Telemetry Path: /var/log/dc_audit.log (and /var/log/sys_audit.log for disk telemetry).

3. Containerized App (Docker)
Network Isolation: The architecture utilizes a dual-bridge network strategy. The wiki service is connected to the frontend network to serve traffic on port 8082, while the db service is placed exclusively on the backend network. This "air-gap" ensures the database is unreachable from the host machine or external network.

Stack Health:

NAME                IMAGE       STATUS         PORTS
titan_backend       mysql:8.0   Up 10 minutes  3306/tcp, 33060/tcp
titan_frontend      nginx       Up 10 minutes  0.0.0.0:8082->80/tcp
 Executive Summary
The Hardened Outpost for Titan Small Business Services now operates under a "Zero Trust" perimeter, utilizing an aggressive firewall and key-based SSH authentication to eliminate common brute-force vectors.
 Redundancy and uptime are monitored by automated Python watchdogs that provide real-time telemetry on core infrastructure connectivity.
 Finally, the application stack is architecturally siloed through containerized network isolation, ensuring that critical data assets remain unreachable even if the frontend is compromised.
