# Week 09 — Session Notes
**Operator:** Maurice Ratiff III
**Topic:** The Application Layer — Web Application Security,
SQL Injection, XSS, CSRF, and API Exploitation
**Date:** May 29th 2026

---

## Summary

Week 9 advanced the attack lifecycle from the network and operating
system layers into the application layer, where the majority of
real-world breaches originate. Across three sessions, coursework
covered SQL Injection for database extraction and authentication
bypass, Cross-Site Scripting and Cross-Site Request Forgery for
client-side attacks, and API traffic interception and Broken Object
Level Authorization exploitation using Burp Suite. The cumulative
Take-Home Lab required chaining all three attack classes against a
unified target to complete a full-stack compromise mission. The week
established that application-layer vulnerabilities are not edge cases
but the dominant breach vector in modern cybersecurity, appearing
consistently in the OWASP Top 10 and in the post-incident reports
of the most consequential data breaches on record.

---

## Session 25: The Data Exfiltration
(SQL Injection)

### Summary

Session 25 introduced SQL Injection as the most damaging attack
class on the modern web, capable of bypassing authentication,
extracting entire database contents, and in some configurations
achieving operating system command execution. The session covered
authentication bypass through tautology injection, database schema
enumeration through UNION-based attacks, and targeted data
extraction from a locally deployed vulnerable application.

### Key Concepts

**SQL Injection:** SQL Injection (SQLi) is a vulnerability that
occurs when user-supplied input is incorporated into a database
query without adequate sanitization or parameterization, allowing
an attacker to modify the query's logic. Because web applications
routinely construct SQL queries using data submitted through forms,
URL parameters, and HTTP headers, SQL Injection represents one of
the broadest attack surfaces in web application security (OWASP,
2021).

**Authentication Bypass:** The most immediately exploitable form
of SQL Injection targets login forms that construct authentication
queries by concatenating user input directly into a SQL WHERE clause.
Submitting a payload such as `' OR '1'='1` as the username causes
the query to evaluate as always true, returning the first user in
the database — frequently an administrative account — without
requiring knowledge of a valid password.

**UNION-Based Extraction:** The SQL UNION operator combines the
results of two SELECT queries into a single result set. When an
attacker can inject a UNION SELECT statement into a vulnerable
query, they can append an additional query that retrieves data from
any table in the database, provided the column count and data types
are compatible. This technique enables systematic extraction of
database schemas, table names, column names, and ultimately the
data stored within them (Stuttard & Pinto, 2011).

**Database Schema Enumeration:** Before extracting target data,
an attacker must identify the structure of the database. In MySQL
and MariaDB environments, the `information_schema` database contains
metadata about all other databases, tables, and columns accessible
to the current database user. UNION-based injection against
`information_schema.tables` and `information_schema.columns` reveals
the complete database structure without requiring prior knowledge
of table names.

**Parameterized Queries:** The primary defense against SQL Injection
is the use of parameterized queries, also known as prepared
statements, in which the SQL query structure is defined separately
from user-supplied data. The database driver treats the parameter
as literal data rather than executable SQL, making injection
structurally impossible regardless of the content of the input
(OWASP, 2021).

### Tools Used

| Tool | Purpose |
|------|---------|
| Browser / form fields | Submit SQL injection payloads |
| `' OR '1'='1` pattern | Authentication bypass via tautology injection |
| UNION SELECT | Append attacker-controlled queries to vulnerable requests |
| `information_schema` | Enumerate database structure |
| Burp Suite Repeater | Iterate and refine injection payloads |

### Lab Observations

The session deployed a locally vulnerable employee directory
application and executed a sequential SQL Injection workflow. The
login portal was bypassed using a tautology payload, granting
access without valid credentials. UNION-based injection was then
used to enumerate the number of columns in the active query,
identify string-compatible columns, and extract table names from
`information_schema.tables`. Targeted extraction against the salary
table retrieved the CEO's compensation record, confirming full
database read access. All injection payloads, query responses, and
extracted data were documented in `sqli_report.txt` with annotations
explaining the logic of each attack step.

### Connection to Defensive Practice

SQL Injection has appeared in the OWASP Top 10 consistently since
the list's inception and remains the vulnerability class responsible
for some of the largest data breaches in history, including the
2008 Heartland Payment Systems breach affecting 130 million card
records (OWASP, 2021). Defensive implementation requires parameterized
queries at every database interaction point — input validation and
web application firewalls are compensating controls, not substitutes.
A defender who has executed UNION-based extraction understands why
error messages must never expose database structure to end users,
why stored procedures require the same parameterization discipline
as inline queries, and why database user accounts should follow
least-privilege principles limiting each account to only the tables
and operations required for its function.

---

## Session 26: The Poisoned Browser
(XSS & CSRF)

### Summary

Session 26 shifted the attack target from the server to the user,
introducing Cross-Site Scripting and Cross-Site Request Forgery as
the primary mechanisms through which attackers weaponize a victim's
authenticated browser session. The session covered both Reflected
and Stored XSS for DOM manipulation and session cookie theft, and
the construction of a CSRF attack link that forces unauthorized
transactions without the victim's knowledge.

### Key Concepts

**Cross-Site Scripting (XSS):** XSS is a vulnerability that occurs
when an application includes user-supplied data in its output without
adequate encoding, allowing an attacker to inject executable
JavaScript into pages viewed by other users. Because the injected
script executes in the context of the vulnerable site, it has access
to the site's cookies, local storage, and DOM, and can perform any
action the authenticated user could perform (Stuttard & Pinto, 2011).

**Reflected XSS:** In a Reflected XSS attack, the malicious payload
is embedded in a URL parameter and reflected back to the user in
the server's response. The attack requires the victim to click a
crafted link containing the payload. Reflected XSS is typically
delivered through phishing emails or malicious redirects and affects
only the user who clicks the link.

**Stored XSS:** In a Stored XSS attack, the malicious payload is
submitted to the application and persisted in the server's database
— for example, as a comment, profile field, or message. Every
subsequent user who views the page containing the stored payload
executes the injected script in their browser, making Stored XSS
significantly more impactful than Reflected XSS as it requires no
interaction from the attacker after initial injection (OWASP, 2021).

**Session Cookie Theft:** The `document.cookie` JavaScript property
exposes all non-HttpOnly cookies associated with the current domain.
An XSS payload that reads `document.cookie` and transmits its value
to an attacker-controlled server enables session hijacking — the
attacker can use the stolen session token to authenticate to the
application as the victim without knowing their password.

**Cross-Site Request Forgery (CSRF):** CSRF exploits the fact that
browsers automatically include authentication cookies with every
request to a domain, regardless of which site initiated the request.
An attacker who crafts a page containing a form or image tag that
submits a request to a vulnerable application can force any
authenticated user who visits the page to unknowingly execute an
action — such as a fund transfer or password change — using their
own session credentials (Stuttard & Pinto, 2011).

### Tools Used

| Tool | Purpose |
|------|---------|
| Browser developer tools | Inspect DOM and observe script execution |
| XSS payload (`<script>alert(1)</script>`) | Confirm XSS vulnerability |
| `document.cookie` payload | Extract session cookie via XSS |
| Crafted HTML form | Deliver CSRF attack against vulnerable endpoint |
| Burp Suite | Intercept and analyze request structure |

### Lab Observations

The Reflected XSS exercise identified a search parameter that
reflected unsanitized user input into the page response and confirmed
JavaScript execution using an alert payload. The Stored XSS exercise
injected a cookie-harvesting payload into a comment field, confirmed
the payload persisted in the database by loading the comments page,
and verified that the cookie value was transmitted to the capture
endpoint upon page load. The CSRF exercise analyzed the structure
of a fund transfer request using Burp Suite, confirmed the absence
of CSRF token validation, and constructed an HTML page containing
a hidden form that submitted the transfer request when visited,
demonstrating silent unauthorized transaction execution. All payloads
and their effects were documented in `xss_payloads.txt`.

### Connection to Defensive Practice

XSS and CSRF defenses are implemented at the application development
level and verified through security testing. XSS is prevented
through context-sensitive output encoding — HTML encoding for HTML
contexts, JavaScript encoding for script contexts — and reinforced
through a Content Security Policy header that restricts which scripts
the browser will execute. CSRF is prevented through synchronizer
token patterns, SameSite cookie attributes, and origin header
validation. The HttpOnly cookie attribute prevents JavaScript access
to session cookies, neutralizing the session theft payload
demonstrated in this session. A defender who has executed these
attacks understands precisely which header configurations and
development practices are required to eliminate each vector.

---

## Session 27: The Invisible Logic
(API Security & Burp Suite)

### Summary

Session 27 addressed the reality that modern applications
increasingly deliver their functionality through REST APIs rather
than traditional web pages, and that these APIs frequently implement
authorization controls less rigorously than their web counterparts.
The session introduced Burp Suite as a man-in-the-middle proxy for
intercepting and manipulating API traffic, covered Broken Object
Level Authorization exploitation, and demonstrated automated
parameter brute-forcing using Burp Intruder.

### Key Concepts

**REST API Security:** A REST API (Representational State Transfer
Application Programming Interface) exposes application functionality
through HTTP endpoints that accept and return structured data,
typically in JSON format. APIs power mobile applications, cloud
services, and microservice architectures. Because API traffic is
not rendered visually in a browser, vulnerabilities in API endpoints
are frequently less visible than those in traditional web interfaces
and may receive less security testing attention (OWASP, 2023).

**Burp Suite:** Burp Suite is an integrated web application security
testing platform developed by PortSwigger. Its core component is
an intercepting proxy that sits between the browser and the
application, capturing and allowing modification of every HTTP
request and response. Burp Suite's additional tools include the
Repeater for manual request manipulation, the Intruder for automated
parameter fuzzing, and the Scanner for automated vulnerability
detection (PortSwigger, 2023).

**Broken Object Level Authorization (BOLA):** BOLA, also known as
Insecure Direct Object Reference (IDOR), occurs when an API endpoint
uses a user-supplied identifier — such as a user ID or record number
— to retrieve data without verifying that the requesting user is
authorized to access the object identified. An attacker who can
manipulate this identifier can access any record in the system,
regardless of ownership. BOLA is ranked as the number one API
security risk by OWASP due to its prevalence and impact (OWASP,
2023).

**Burp Intruder:** The Burp Intruder tool automates the submission
of a modified HTTP request with a defined set of payloads inserted
at a specified position. This capability enables brute-force attacks
against hidden parameters, discovery of valid identifiers through
enumeration, and automated testing of input validation controls.

### Tools Used

| Tool | Purpose |
|------|---------|
| Burp Suite Proxy | Intercept and inspect live API traffic |
| Burp Suite Repeater | Manually modify and resubmit API requests |
| Burp Suite Intruder | Automate parameter brute-forcing |
| Browser proxy configuration | Route traffic through Burp Suite |
| JSON response inspection | Identify authorization failures in API responses |

### Lab Observations

The session configured the browser to route traffic through Burp
Suite's intercepting proxy, capturing API requests generated by
the target application. The BOLA exercise identified an API endpoint
that returned user profile data based on a numeric user ID in the
request path. Modifying the ID value in Burp Repeater to reference
the CISO's account ID returned the CISO's private profile data
without authentication error, confirming the authorization control
failure. The Burp Intruder exercise configured a discount code
parameter as the injection point, loaded a wordlist of candidate
codes, and ran the attack, identifying the valid discount code
through response length differentiation. All findings were
documented in `api_audit.log`.

### Connection to Defensive Practice

API security testing is an increasingly critical component of
application security programs as organizations migrate functionality
from web interfaces to API-driven architectures. BOLA is
particularly dangerous because it requires no special tools or
techniques — any authenticated user can test for it by modifying
an ID value in a request. Defensive implementation requires
object-level authorization checks at every API endpoint, verifying
that the authenticated user has permission to access the specific
object requested — not just that they are authenticated. Burp Suite
is used by defensive teams as well as offensive practitioners:
security engineers use it during code review and pre-release testing
to verify that authorization controls are implemented correctly
before an application reaches production.

---

## TLAB 9: Operation Omni-Portal
(Full-Stack Web Compromise)

### Summary

The Week 9 Take-Home Lab required chaining SQL Injection, Stored
XSS, and API BOLA vulnerabilities in sequence against the Titan
Omni-Portal to exfiltrate confidential order data. No single
vulnerability provided a complete path to the target data; each
technique was required to advance the attack chain to the next
stage. The completed assessment was documented in
`OmniPortal_Assessment.md`, providing a structured record of each
exploitation step, the vulnerability exploited, and the data
accessed at each stage.

### Connection to Defensive Practice

The chained exploitation structure of Operation Omni-Portal reflects
the reality of sophisticated web application attacks, in which
attackers combine multiple lower-severity vulnerabilities to achieve
an impact that none of the individual vulnerabilities would enable
alone. Defensive programs that test for and remediate individual
vulnerability classes in isolation may remain exposed to chained
attacks that traverse multiple components. A comprehensive
application security program requires end-to-end attack simulation
— penetration testing that attempts to chain vulnerabilities across
authentication, data access, and client-side controls — to identify
composite risks that component-level testing misses.

---

## References

OWASP. (2021). *OWASP Top 10 — 2021*.
    https://owasp.org/Top10/

OWASP. (2023). *OWASP API security top 10 — 2023*.
    https://owasp.org/API-Security/editions/2023/en/0x11-t10/

PortSwigger. (2023). *Burp Suite documentation*.
    https://portswigger.net/burp/documentation

Stuttard, D., & Pinto, M. (2011). *The web application hacker's
    handbook: Finding and exploiting security flaws* (2nd ed.).
    Wiley.
