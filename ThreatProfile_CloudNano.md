# TARGET THREAT PROFILE: CloudNano 
**Classification:** Passive Security Audit
Maurice Ratiff III - Operator on Hand

 1. Subdomain Discovery 
* **Tool Used:** Sublist3r
* **Subdomains Found:**
 Found over 40 Subdomains: mfa.tesla.com potentially indicating subdomains for multi-factor authentication. 
 Second subdomain found was sso.tesla.com this indicates where single - sign on would operate and log in for users.

## 2. Tech Stack Mapping 
* Tool Used for Tech Stack Mapping was BuiltWith.com
* **Identified Technologies (CMS/CDN/Backend):** 
Backend Technologies that were discovered which how the site was built were programming languages Python and Go due to the 
indication of Fleet Application programmable interfaces.
 
Content Delivery Network that was used within the tesla website that was Akamai M Pulse which is a Real User Monitoring that 
monitors performance and business metrics of user data which would compose of user product searches and behaviors such as 
particular tesla model cars. Each query pings a node within the monitoring tool. If queries are unsecure and open threat actors
can lead and acquire sensitive data for confidential business metric information.

## 3. Major Exposure Points & Dangers 
*(List three major exposure points discovered during your OSINT audit and explain why they are dangerous)*
1. Exposure Point 1 would be the developer tesla subdomain usually these domains are typically not properly security hardened
which can lead to major credential exploits due to not being monitored closely.

2. Toolbox.tesla.com which holds important diagnostic tools for the site if these are leaked important processes pertaining to 
running diagnostic to check on potential exposures and vulnerabilities can allow threat actors to understand patterns in updates and 
security process on the tesla site.

 
3. logcollection.tesla.com a major gold mine for an attacker if they can gain access and find out where confidential files are being sent they can then identify 
security tools and processes that are in place. 
