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
* **Host-Based Lockdown:** Im
