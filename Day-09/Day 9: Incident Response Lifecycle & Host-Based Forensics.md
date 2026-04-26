# Day 9: Incident Response Lifecycle & Host-Based Forensics

**Objective:** Execute the full NIST Incident Response (IR) Lifecycle (Preparation, Detection & Analysis, Containment, Eradication, and Recovery). This module focuses on safely identifying a malicious artifact, establishing cryptographic evidence integrity, correlating local data with global threat intelligence, isolating the compromised host, and auditing for hidden persistence mechanisms.

**Mindset Shift:** Moving from "Auditing a static system" to "Active threat neutralization." Today's focus is acting as a **First Responder (SOC Analyst)** where speed, precise documentation, and strict adherence to the Chain of Custody are required to stop an active breach without destroying critical forensic evidence.

**Lab Environment:**
* **OS:** Linux Environment (pulse-HP-Notebook).
* **Tools:** Native Linux CLI (`sha256sum`, `crontab`, `ufw`, `ps`, `file`, `kill`), VirusTotal (Threat Intelligence Correlation).

---

### 🕒 Hour 1: Evidence Integrity & Cryptographic Baselining (Preparation & Detection)

The objective was to identify a suspicious file and secure it cryptographically before any analysis occurred to preserve the legal Chain of Custody.
* **Artifact Generation:** Created a simulated malicious artifact (`sample_malware.txt`) utilizing the industry-standard EICAR antivirus test string via the `echo` command.
* **Initial Interrogation:** Verified file properties, permissions, and existence using `ls -l` and the `file` command to look past potentially deceptive file extensions.
* **Cryptographic Fingerprinting:** Before manipulating the file, generated a hash using `sha256sum sample_malware.txt`. This provided a unique mathematical signature (starting with `131f95c...`), ensuring **Data Integrity**. If even one byte of the file is altered during investigation, the hash will change, proving evidence tampering.

### 🕒 Hour 2: Threat Intelligence Correlation (Analysis)

Rather than executing the file to see what it does, I transitioned from local identification to global intelligence correlation to understand the threat safely.
* **Precision Querying:** Queried the VirusTotal database. *Lesson Learned:* Initial searches combining the hash and filename returned errors. I refined the search to utilize *only* the SHA-256 hash, as threat intelligence databases index by cryptographic signatures, not localized filenames.
* **Vendor Signature Analysis:** Analyzed the vendor breakdown, identifying a conclusive 62/67 malicious detection rate across global security engines.
* **Threat Identification:** Successfully identified the specific malware family (EICAR) and analyzed associated Threat Actor behavior flags (e.g., known-distributor traits) to understand the nature of the simulated threat without detonating it locally.

### 🕒 Hour 3: Host Isolation & Egress Filtering (Containment)

With the threat confirmed, the immediate priority was to stop lateral movement (infecting other machines) and sever any potential Command & Control (C2) communication from the attacker.
* **Host-Based Lockdown:** Implemented strict network containment using the Uncomplicated Firewall (`ufw`).
* **Policy Execution:** Applied a "Default Deny" architecture by executing `sudo ufw default deny incoming` and `sudo ufw default deny outgoing`, followed by enabling the firewall. This keeps the machine powered on for live memory forensics while blinding the attacker.
* **Mathematical Verification:** It is not enough to apply a rule; it must be verified. I proved successful egress filtering by attempting to ping an external IP (`google.com`), resulting in 100% packet loss and temporary name resolution failure.

### 🕒 Hour 4: Persistence Hunting & Process Termination (Eradication)

A core tenet of Incident Response is that attackers rarely leave after dropping a file; they establish "Persistence" to survive system reboots. This phase focused on finding the roots of the infection.
* **User-Level Audit:** Conducted a persistence audit by checking for unauthorized scheduled tasks using `crontab -l` for the current user.
* **System-Level Audit:** Expanded the threat hunt to system-wide directories by analyzing `/etc/cron.*` (daily, weekly, monthly) for hidden, root-level backdoors.
* **Process Mapping & Termination:** Documented the methodology for tracking the execution source via the process tree (`ps -auxf | grep -i "sample"`) to identify parent-child process relationships, and forcefully terminating malicious PIDs using `kill -9`.
* **Secure Deletion:** Executed the final removal of the physical artifact (`rm sample_malware.txt`).

### 🕒 Hour 5: Service Restoration & Post-Incident Review (Recovery)

With the threat eradicated and persistence mechanisms cleared, the system was safely transitioned back to an operational state.
* **Containment Removal:** Disabled the network lockdown (`sudo ufw disable`) and restored verified core services.
* **Executive Documentation:** Drafted the formal Incident Report categorizing the attack using the **MITRE ATT&CK Framework**:
    * *Initial Access:* T1566 (Phishing)
    * *Execution:* T1059 (Command and Scripting Interpreter)
* **Monitoring:** Established a protocol for enhanced logging and close-watch monitoring on the host for the subsequent 24-48 hours to ensure no secondary beacons activated.

### 🕒 Hour 6: Enterprise Scaling & Tooling Strategy (Lessons Learned)

A security professional must apply single-host lessons to an enterprise environment. I documented the strategic remediation required to automate this workflow across thousands of endpoints.
* **EDR Deployment:** Formulated a transition strategy from manual CLI forensics to using an Endpoint Detection and Response (EDR) platform (such as the open-source Wazuh) to automate the detection-to-containment pipeline.
* **Architecture Hardening:** Documented recommendations for the network team, including the implementation of strict File Execution Prevention policies and advanced Security Awareness Training focused on identifying double-extension obfuscation (e.g., `.txt.exe`).

---

### 📸 Evidence & Artifacts
*(Visual proof of the investigation, correlating directly to the incident timeline).*
**Practical evidence** [View scan_logs.sh Script & Execution Screenshots](Evidence/)

 
| Artifact Name | Description |
| :--- | :--- |
| `IR_D9_01_SHA256_Hash_Generation.png` | Documentation of evidence integrity. Generated a SHA-256 fingerprint in the Linux CLI prior to analysis. |
| `IR_D9_04_VT_Global_Detection_Summary.png` | Threat intelligence verification via VirusTotal API showing a 62/67 engine detection rate for the artifact. |
| `IR_P3_Network_Isolation_UFW.png` | Host-based containment implemented via UFW, showing active `deny incoming/outgoing` policies. |
| `IR_P3_Containment_Verification_Ping.png` | Proof of successful network isolation. Verified egress filter functionality via 100% packet loss on external ping. |
| `IR_P4_Persistence_Audit_Crontab.png` | Evidence of the Eradication phase. Audited user-level scheduled tasks to ensure no backdoors existed. |
| `IR_P5_Recovery_Firewall_Restore.png` | Final recovery phase returning the system to operational status by safely lifting the firewall lockdown. |

---

### 🧠 Daily Reflection

Today shifted my focus from finding vulnerabilities to actively managing a crisis. I learned that simply deleting a malicious file is the mark of an amateur; a professional ensures a mathematically proven Chain of Custody (Hashing), correlates findings with global intelligence, strictly contains the network before taking action, and aggressively hunts for hidden persistence mechanisms (Crontab). By executing the NIST Incident Response lifecycle manually via the command line, I now deeply understand the foundational mechanics that enterprise EDR tools (like Wazuh or CrowdStrike) automate. My focus is no longer just on "breaking things" or "finding the bug"—it is on forensic containment, data integrity, and restoring business continuity.
```
