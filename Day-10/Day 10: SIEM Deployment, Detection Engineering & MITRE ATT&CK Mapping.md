## **Day 10: SIEM Deployment, Detection Engineering & MITRE ATT&CK Mapping**

### **The Mindset Shift: From "Exploitation" to "Continuous Detection"**
Today’s focus represented a critical transition from an offensive mindset ("How do I break in and escalate?") to a defensive, Detection Engineering methodology ("How do I ensure no lateral movement or escalation goes unnoticed?"). Moving from point-in-time penetration testing to continuous, telemetry-driven monitoring requires anticipating adversary behaviors and building automated, high-fidelity tripwires for persistence and credential dumping techniques.

---

## 🕒 Hour-by-Hour Technical Execution

### **Hour 1-2: Telemetry Ingestion & SIEM Pipeline Configuration**
* **Goal:** Establish a robust log ingestion pipeline to monitor host-level authentication and authorization events.
* **Action:** Configured Splunk Enterprise to actively monitor Linux system logs.
    ```bash
    # Setting up the data inputs for local host monitoring
    sudo tail -f /var/log/auth.log
    ```
* **Discovery:** Validated that the Universal Forwarder/local ingestion engine was successfully normalizing `auth.log` data, providing visibility into `sudo` execution and session initiations.

### **Hour 3-4: Detection Engineering & Alert Automation**
* **Goal:** Translate raw log telemetry into automated security intelligence by filtering out administrative noise.
* **Action:** Developed specific Search Processing Language (SPL) queries to hunt for high-risk indicators and tied them to automated Cron schedules (`*/1 * * * *`).
    ```splunk
    # SPL query for detecting SUID bit manipulation (Privilege Escalation)
    index=main "chmod" ("+s" OR "4777" OR "u+s")
    ```
* **Discovery:** Successfully established a real-time polling metric. The SIEM is now configured to trigger high-severity notifications upon detecting specific command-line arguments.

### **Hour 5: Adversary Simulation & Audit Validation**
* **Goal:** Execute controlled Red Team simulations to validate the Blue Team detection pipeline and ensure audit accountability.
* **Action:** Simulated MITRE ATT&CK techniques T1548.001 and T1003.008, followed by a system sanitization and audit check.
    ```bash
    # Simulating T1548.001 (SUID Persistence)
    sudo chmod u+s /tmp/not_malware.sh
    
    # Simulating T1003.008 (Credential Dumping)
    sudo cat /etc/shadow
    
    # SIEM shutdown and resource release
    sudo /opt/splunk/bin/splunk stop
    ```
* **Discovery:** The automated alerting engine successfully caught both the SUID manipulation and the unauthorized credential store access. Furthermore, the `index=_audit action=edit_alerts` query verified that the eventual disabling of the lab alerts was formally tracked by the system.

---

## 📊 Technical Analysis: The "Big Three" Concepts

| Concept / Category | Definition & Technical Focus | Implementation / Observation |
| :--- | :--- | :--- |
| **T1548.001: SUID/SGID Abuse** | Adversaries manipulate file permissions to execute binaries with the privileges of the file owner (often root) for persistence. | Captured via continuous monitoring of `chmod` commands containing `u+s` or `4777` arguments. |
| **T1003.008: /etc/shadow Access** | The targeted extraction of local credential hashes for offline brute-force or dictionary cracking. | Monitored by tracking any file read operations against the `/etc/shadow` directory. |
| **Governance: Policy vs. Standard** | **Policy:** A general mandatory rule (e.g., "All authentication events must be logged to the SIEM"). <br><br> **Standard:** A specific metric-based rule (e.g., "The SIEM must poll `auth.log` data using a Cron schedule of `*/1 * * * *`"). | Enforced by establishing the ingestion pipeline (Policy) and creating the 60-second automated alert triggers (Standard). |

---

## 📂 Evidence & Artifacts Gallery

| Artifact Name | Description | Technical Observation |
| :--- | :--- | :--- |
| `Day10_H2_Data_Source_Verification.png` | Splunk Data Ingestion Check | Proves the successful pipeline configuration, showing live event streaming from the `pulse-HP` host. |
| `Day10_H4_SOC_Incident_Dashboard.png` | Triggered Alerts Console | Validates the Detection Engineering phase; shows automated alerts firing based on the programmed Cron logic. |
| `Day10_H5_T1548_Detection_Proof.png` | SUID Manipulation Log | Raw telemetry proving the SIEM captured the exact `chmod u+s` adversary simulation. |
| `Day10_H5_Critical_Shadow_File_Alert.png` | Credential Access Trigger | High-fidelity event log capturing the execution of `cat /etc/shadow`, indicating early-stage lateral movement prep. |
| `Day10_H5_System_Integrity_Audit.png` | Splunk Internal Audit Trail | Shows the `_audit` index accurately tracking the manual disabling of alerts, proving environmental accountability. |

---

## 🛡️ Remediation & Hardening Strategy

Based on today's simulated threats and detection validations, the following mitigation steps are recommended to harden the host environment:

1.  **SUID Binary Auditing:** Implement automated scripts to run `find / -perm -4000 -type f 2>/dev/null` weekly. Compare the output against a known-good baseline to identify rogue SUID implementations.
2.  **Strict File Permissions (Credential Stores):** Ensure `/etc/shadow` permissions remain strictly `640` or `000` depending on the OS architecture, owned by `root:shadow`. 
3.  **SIEM Alert Tamper Protection:** Configure Role-Based Access Control (RBAC) within the SIEM to ensure that Tier 1 Analysts cannot disable or mute critical alerts without Manager-level approval.
4.  **Implement 'Least Privilege' for Sudoers:** Review the `/etc/sudoers` file to ensure users are only granted specific command execution rights rather than blanket `ALL=(ALL:ALL) ALL` access, limiting the damage of compromised accounts.

---

## 📝 Daily Reflection

**Problem:** Standard system logging natively captures a massive amount of noise, making it highly difficult to retroactively identify stealthy privilege escalation or data dumping during an active incident. 

**Action:** Engineered a focused detection pipeline in Splunk, utilizing specific SPL queries to isolate T1548 and T1003 behaviors, and tied them to automated, metric-based polling standards for near real-time notification.

**Result:** The pipeline successfully captured the simulated Red Team behavior. More importantly, from a forensic auditing perspective, the environment demonstrated verifiable remediation. Not only were the malicious actions detected, but the subsequent administrative actions to sanitize the lab (disabling the alerts) were tracked within the internal audit index, proving that the system maintains absolute integrity and a continuous chain of custody for all configurations.
