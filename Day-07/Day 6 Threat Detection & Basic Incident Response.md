**Day 7: Web Application Penetration Testing & Vulnerability Assessment**

**Objective:** Transition from defensive threat detection and log monitoring to active offensive security operations. This module focuses on mapping a web application's attack surface, identifying critical vulnerabilities within the OWASP Top 10 framework, executing SQL Injection and Cross-Site Scripting (XSS) exploits, and achieving full administrative compromise through privilege escalation and authorization bypass.

**Mindset Shift:** Moving from "How do I defend this system?" to "How can I bypass these controls?" Today’s focus is acting as a **Security Consultant** to discover, document, and analyze architectural weaknesses before they can be exploited by malicious actors. Vulnerabilities are no longer just bugs—they are logical entry points into the core of an enterprise.

**Lab Environment:**
* **VirtualBox hosting Parrot Security OS (Linux Mint Environment)**
* **OWASP Juice Shop (Vulnerable Web Application Platform)**
* **Browser DevTools & Terminal-based Security Tools (Nmap, DIRB)**
* **SQL Injection Payloads & Custom Wordlists**

**🔎 Phase 1: Identifying Information Leaks & Attack Surface Mapping**

The primary objective was to hunt for sensitive data exposure and framework vulnerabilities through both passive and active browser-based enumeration, identifying how the application handles data before any tools were even launched.

**Technology Stack & Framework Profiling**

By inspecting the Document Object Model (DOM) and analyzing the response headers, I identified the specific technology stack powering the application.
* **Technical Discovery:** Isolated the framework as **Angular version 20.3.18**.
* **Analysis:** Identifying specific version numbers is a critical step in the reconnaissance phase. It allowed me to cross-reference known CVEs (Common Vulnerabilities and Exposures) associated with this specific Angular build, revealing that older or unpatched versions often harbor vulnerabilities related to client-side sanitization.

**Sensitive Data Exposure (The Admin Leak)**

I audited public-facing reviews, customer feedback sections, and FAQ pages to check for PII (Personally Identifiable Information) leaks and metadata exposure.
* **Finding:** The application failed to mask user identities in the product review section, explicitly exposing the primary administrative email: **admin@juice-sh.op**.
* **Analysis:** This discovery provided the "Target ID" for the next phase. In a real-world scenario, this email would be the primary target for credential stuffing, brute force attacks, or targeted phishing (Whaling) to gain entry into the management tier.

**🌐 Phase 2: Infrastructure Auditing & Directory Discovery**

Every hidden directory or unlinked file is a potential doorway. My objective was to use automated tools to uncover the "hidden" side of the server and map out the file system architecture.

**Network Service Enumeration (Nmap)**

Using Nmap to identify open services, port status, and server fingerprints to understand the hosting infrastructure.
* **Command Executed:** `nmap -sV juice-shop.herokuapp.com`
* **Analysis:** The scan confirmed that ports **80 (HTTP)** and **443 (HTTPS)** were open and operating behind a **Heroku-router**. This fingerprint established the perimeter boundaries, indicating that the application is likely containerized, which influences how an attacker might attempt to pivot within the internal network.

**Directory Brute-Forcing (DIRB)**

Using automated wordlists to find folders and files that are not linked in the main user interface.
* **Troubleshooting:** Encountered repository failures when fetching standard wordlists. I successfully pivoted by using `wget` to source a manual wordlist from a remote repository, ensuring the engagement could continue despite local environment hurdles.
* **Command Executed:** `dirb http://juice-shop.herokuapp.com/ ./common.txt -N 503`
* **Analysis:** By applying the **-N 503** flag, I filtered out the "Service Unavailable" noise returned by the Heroku firewall. This allowed me to uncover sensitive, high-value directories including **/ftp** (likely for file storage), **/Video**, and the restricted **/administration** panel.

**🔗 Phase 3: Vulnerability Exploitation — "The Kill Chain"**

Correlation is the hallmark of an expert analyst. By combining the administrative email discovered in Phase 1 with the hidden login portal identified in Phase 2, I executed the following attack timeline to breach the system.

**Authentication Bypass (SQL Injection)**

I targeted the login portal to test how the backend database handles manipulated queries and whether it uses prepared statements.
* **Payload Used:** `' OR 1=1 --`
* **Analysis:** This payload manipulated the SQL query logic on the backend. Because the application concatenated user input, the payload forced the database to evaluate the statement as **True** for the first user record. I successfully bypassed authentication and was granted full access as the **Administrator** without possessing a password.

**Client-Side Execution (Reflected XSS)**

Testing the search bar for improper output encoding to see if user-supplied scripts are executed by the browser.
* **Payload Used:** `<iframe src="javascript:alert('Hacked')">`
* **Analysis:** The application failed to sanitize the input, rendering the script directly in the browser's DOM. This execution proved that an attacker could inject malicious JavaScript to steal session cookies, capture keystrokes, or redirect users to a malicious site.

**🛡️ Phase 4: Privilege Escalation & Incident Simulation (Containment)**

Upon confirming the compromise, I navigated to the hidden administrative dashboard found earlier to assess the maximum impact of the breach.

**Administrative Compromise**
* **Critical Finding:** Accessed the **User Management table**, which allowed for full visibility of all registered users and their PII. More critically, the interface provided the authority to arbitrarily delete user feedback and potentially modify account statuses.
* **Impact Analysis:** This represents a total loss of **Confidentiality** and **Integrity** for the application database.
* **Containment Recommendation:** To sever the attacker's ability to manipulate logic, the development team must implement **Parameterized Queries** immediately and enforce server-side role validation for all `/admin` routes.

**🤖 Phase 5: Mitigation & Secure Architecture Recommendations**

A security professional must provide a clear path to recovery. I drafted the following remediation strategy based on the OWASP Top 10:
* **Parameterized Queries (Prepared Statements):** All database interactions must treat user input as data, never as executable code, to eliminate SQL Injection risks.
* **Output Encoding:** Implement strict, context-aware sanitization to encode data (e.g., converting `<` to `&lt;`) before it is rendered in the browser.
* **RBAC (Role-Based Access Control):** Enforce strict server-side authorization checks for all administrative endpoints; knowing a URL should never be the only requirement for access.

**📸 Evidence & Artifacts**

**1. Passive Reconnaissance (PII Leak)**
* **Description:** Documentation of the administrative email address found via unprotected product reviews.

**2. Active Scanning (Network Discovery)**
* **Description:** Nmap service scan results identifying the cloud-hosting architecture and open web ports.

**3. Exploitation (The SQLi Bypass)**
* **Description:** Successful proof-of-concept showing a logged-in administrative session achieved via the `' OR 1=1 --` payload.

**4. Privilege Escalation (The Smoking Gun)**
* **Description:** Screenshot of the unauthorized access to the restricted **Administrative Panel**, demonstrating full control over the application's user data.

**🧠 Daily Reflection**

Today marked the tipping point between being a system administrator and becoming a cybersecurity professional. By learning to correlate a simple email leak with an open directory and an unsanitized input field, I practically applied the **"Kill Chain"** methodology. It is now clear that individual vulnerabilities are often just noise; real security value is derived from understanding the entire application lifecycle and documenting how isolated flaws can be chained together to achieve a total system compromise. My next focus will be on the **Remediation** phase—ensuring these doors stay closed once they are identified.
