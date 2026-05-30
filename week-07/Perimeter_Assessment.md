# TITANCORP: PERIMETER ASSESSMENT REPORT
Operator Maurice Ratiff III the subnet is  172.88.0.0/24

## PHASE 1: ACTIVE ENUMERATION (NMAP)
*(List the live IPs discovered and their running services/versions)*
Host 1 (172.88.0.10) the service name and version is  http nginx 1.14.2
Host 2 (172.88.0.15) the service name and version is All 1000 scanned ports on syn-172-088-000-015.res.spectrum.com (172.88.0.15) are in ignored states.
Host 3 (172.88.0.20) the service name and version is http Apache httpd 2.4.66 ((Unix))

## PHASE 2: VULNERABILITY AUDIT (NIKTO)
*(Run Nikto against the TWO web servers discovered above. List one major finding for each.)*
 For the Web Server 172.88.0.10 one finding I found is that the header to prevent anti-clickjacking was not present or discovered
 which indicates vulnerability to clickjacking. Also, server leakes inodes etags that have no present http headers indicating http headers being exposed.
Nginx service is extremely outdated as well needs to be shut down and deem a legacy system.

Web Server 172.88.0.20 Cross -site tracing attack using the HTTP TRACE METHOD due to finding a vulnerability that has anti-clickjacking not being enabled and the HTTP Trace 
Method being enable. Which indicates that XST has a high possibility to happen. 


## PHASE 3: RISK TRIAGE
*(Review your findings. Identify the SINGLE highest-risk vulnerability across the entire DMZ. Justify why it is the top priority using the Likelihood x Impact formula.)*

* **Top Priority Remediation:** Finding number two web server IP Address 172.88.0.20 using the service Apache httpd 2.4.66((Unix)) using http. HTTP Trace Method is active 
indicating that the web server has a Cross - Site Tracing Vulnerability which can be exploited and a Cross - Site Tracing Attack can commence.
* **Justification:** Likehood High x Impact High due to the trace method being able to bypass the httponly cookie which then a threat actor can use to steal a user's credentials 
the method TRACE requires no authentication which can allow for a simple bypass without any extensive multi-authentication factors.
TRACE allows the client to see what is being received at the other end of the request chain and use that data for testing or diagnostic information, can then use steal user's cookies and 
session hijack to steal user credentials(Bank information).
