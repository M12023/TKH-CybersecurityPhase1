# CLOUDNANO REMEDIATION PLAN
Maurice Ratiff III - Operator 
TOP 5 CRITICAL FIXES
*(From the 20 raw findings, select the 5 that pose the greatest ACTUAL risk. Explain your reasoning.)*

1.  Unauthenticated AWS S3 Bucket Likelihood: Extremely High (Publicly accessible/Automated scanners) / Impact: Critical (Direct leak of Customer PII, massive legal/GRC failure).

2.  SQL Injection in Login Page Likelihood: High (Common attack vector on database portals) / Impact: High (Massive exfiltration of customer credentials and database records).


3. Remote Code Execution (Apache Struts) Likelihood: High (Internet-facing system) / Impact: Critical (Total server takeover and foothold for lateral movement).

4. SMBv1 Enabled on HR File Server Likelihood: Medium (Internal network, but trivial to exploit once inside) / Impact: High (HR data theft and likely entry point for Ransomware).


5. Cross-Site Scripting (XSS) Likelihood: High (Public support forum) / Impact: Medium (Session hijacking of forum users, though less critical than direct DB access).
