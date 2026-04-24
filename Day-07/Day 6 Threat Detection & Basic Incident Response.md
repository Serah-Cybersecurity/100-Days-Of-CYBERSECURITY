Day 7: Web Application Penetration Testing & Vulnerability Assessment

**Objective:** Transition from defensive threat detection and log monitoring to active offensive security operations. This module focuses on mapping a web application's attack surface, identifying critical vulnerabilities within the OWASP Top 10 framework, executing SQL Injection and Cross-Site Scripting (XSS) exploits, and achieving full administrative compromise through privilege escalation.

**Mindset Shift:** Moving from "How do I defend this system?" to "How can I bypass these controls?" Today’s focus is acting as a **Security Consultant** to discover and document architectural weaknesses before they can be exploited by malicious actors.

**Lab Environment:**
* **Target:** OWASP Juice Shop (Heroku-hosted)
* **Operating System:** Parrot Security OS (Linux Mint Environment)
* **Tooling:** Nmap, DIRB, Browser DevTools, SQL Injection Payloads

---

🔎 **Phase 1: Passive Reconnaissance & Manual Enumeration**

Reconnaissance is the foundation of any successful engagement. My objective was to map the application's "front-of-house" logic and identify sensitive information leaks without triggering security alerts.

**Framework & Version Profiling**
By inspecting the Document Object Model (DOM), I identified the specific technology stack powering the application.
* **Technical Discovery:** Isolated the framework as **Angular version 20.3.18**.
* **Command/Action:** Utilized Browser DevTools (F12) to audit the source code tags.
* **Analysis:** Identifying specific version numbers is critical for "Zero-Day" research. This allowed me to cross-reference known CVEs (Common Vulnerabilities and Exposures) associated with this specific Angular build.
* **Artifact:** `Day 7 -Framework version Leak-Angular version (20.3.18).png`

**Sensitive Data Exposure (The Admin Leak)**
I audited public-facing reviews and feedback sections to check for PII (Personally Identifiable Information) leaks.
* **Finding:** The application failed to mask user identities in reviews, explicitly exposing the administrative email: `admin@juice-sh.op`.
* **Analysis:** This provided the "Target ID" for the next phase, significantly narrowing the focus for credential-based attacks.
* **Artifact:** `Day 7 ADMIN EMAIL LEAK-Admin Email (admin@juice-sh.op)..png`

---

🌐 **Phase 2: Active Scanning & Infrastructure Analysis**

Once the attack surface was mapped, I moved to active interrogation of the server infrastructure to find hidden entry points using command-line tools.

**Network Service Enumeration (Nmap)**
Using Nmap to identify open services and server fingerprints to understand the hosting environment.
* **Command Executed:** `nmap -sV juice-shop.herokuapp.com`
* **Analysis:** Confirmed ports 80 (HTTP) and 443 (HTTPS) were open on a Heroku router. Identifying the router fingerprint helps in understanding perimeter defenses and potential timeout behaviors.
* **Artifact:** `DAY 7 -nmap service scan.png`

**Directory Brute-Forcing & Troubleshooting (DIRB)**
Using automated wordlists to find "hidden" directories not linked in the main UI.
* **The Troubleshooting Process:** Encountered a repository failure where the system could not locate the standard `wordlists` package.
* **Resolution:** Successfully pivoted by using `wget` to download a custom wordlist directly into the working directory.
* **Command Executed:** `dirb http://juice-shop.herokuapp.com/ ./common.txt -N 503`
* **Analysis:** By applying the `-N 503` flag, I filtered out the "Service Unavailable" noise from the Heroku firewall, uncovering sensitive directories including `/ftp`, `/Video`, and the high-value `/administration` panel.
* **Artifact:** `Day 7-Wordlist_Download_Troubleshooting.png`

---

🔗 **Phase 3: Vulnerability Exploitation — "The Kill Chain"**

Using the intelligence gathered, I executed a multi-stage attack to bypass security controls and prove the impact of poor input sanitization.

**Authentication Bypass (SQL Injection)**
I targeted the login portal to test how the backend database handles manipulated queries.
* **Payload Used:** `' OR 1=1 --`
* **Analysis:** This "Classic SQLi" payload forced the database to evaluate the login statement as `True` regardless of the password. By injecting this into the email field, I broke the logic of the SQL query and was granted access as the first user in the table—the Administrator.
* **Artifact:** `Day 7-SQL_Injection_Bypass.jpg`

**Client-Side Execution (Reflected XSS)**
Testing the search bar for improper output encoding to execute scripts in the victim's browser.
* **Payload Used:** `<iframe src="javascript:alert('Hacked')">`
* **Analysis:** The application rendered the script directly in the browser's DOM. This proved that an attacker could execute malicious JavaScript to steal session cookies or perform actions on behalf of other users.
* **Artifact:** `Day 7-XSS_Success_Notification..png`

---

🛡️ **Phase 4: Privilege Escalation & Impact Analysis**

Upon gaining initial access, the objective was to demonstrate the depth of the compromise and the potential for lateral movement.

**Broken Access Control (Administrative Compromise)**
By manually navigating to the `/administration` endpoint while authenticated via the SQLi bypass, I tested for server-side authorization checks.
* **Critical Finding:** The application relied on "Security through Obscurity" rather than robust authorization. I gained full access to the User Management table.
* **Impact:** From this dashboard, I could view every registered email address and had the authority to delete user profiles and feedback at will.
* **Artifact:** `Day 7-Admin_Panel_Privilege_Escalation.png`

---

🤖 **Phase 5: Mitigation & Remediation Recommendations**

To secure the application, I drafted the following professional remediation strategy:
1.  **Implement Parameterized Queries:** To prevent SQL Injection, all database queries must treat user input as data, not executable code.
2.  **Context-Aware Output Encoding:** To prevent XSS, all user-supplied data must be sanitized/encoded before being rendered in the browser.
3.  **Enforce Server-Side Authorization:** Access to administrative endpoints must be verified via secure session tokens and role-based permissions (RBAC), regardless of whether the URL is "hidden."

---

📸 **Evidence & Artifacts**

* **Reconnaissance:** `Day 7 -Attack surface overview.png`
* **Directory Discovery:** `Day 7-DIRB_Filter_503_Errors.png`
* **Vulnerability Proof:** `Day 7-XSS_Reflected_Payload.png`
* **Environment Setup:** `Day 7 - DIRB_Installation_Error_Log.png`

---

**🧠 Daily Reflection**

Today was a major milestone in my cybersecurity pivot. By successfully executing the "Kill Chain"—moving from a simple email leak to full administrative control—I learned that security is only as strong as its weakest link. It became clear that while automated tools like DIRB and Nmap are essential, the real value of a security professional lies in the ability to correlate small leaks (like a framework version or a public email) into a successful exploitation strategy. This "Consultant Mindset" will be the focus of my portfolio development as I move toward more complex network environments.
