# Day 6: Threat Detection & Basic Incident Response

**Objective:** Transition from passive log monitoring to active threat hunting. This module focuses on identifying Indicators of Compromise (IoCs), auditing network anomalies, correlating cross-platform events to build an attack timeline, and executing basic incident containment.

**Mindset Shift:** Logs and processes are no longer just data—they are clues. Today’s focus is acting as a "Digital Detective" to connect isolated events into a cohesive narrative of an attack.

**Lab Environment:**
* VirtualBox hosting Parrot OS (Linux)
* Windows Host (PowerShell)
* Simulated Attack Vectors (Brute Force, Reverse Shells)

---

## 🔎 Phase 1: Identifying Indicators of Compromise (IoCs)

An **Indicator of Compromise (IoC)** is digital forensic evidence that a system has been breached or is currently under attack. My objective was to hunt for anomalous authentication behaviors across both operating systems.

### Linux Threat Hunting (Brute Force Detection)
Attackers often use automated scripts to guess passwords. Instead of just finding failed logins, I used command-line text parsing to count the *frequency* of attacks from specific IP addresses.

**Command Executed (via Journalctl):**
```bash
# Extracting failed SSH root login attempts, sorting by IP, and counting the frequency
sudo journalctl | grep "Failed password" | awk '{print $11}' | sort | uniq -c
```
*Analysis:* This command isolated the target IP (`192.168.1.108`) that was actively attempting to breach the `root` account, proving targeted reconnaissance rather than a random error.

### Windows Threat Hunting (Explicit Credentials)
On Windows, attackers who compromise a standard user account will often attempt to use the `runas` command or script to execute payloads as an Administrator. This triggers Event ID **4648**.

**Command Executed (PowerShell):**
```powershell
# Querying the Security log specifically for the use of explicit credentials
Get-WinEvent -FilterHashtable @{LogName='Security'; ID=4648} -MaxEvents 5
```
*Analysis:* Identifying Event 4648 is critical because it highlights potential **Privilege Escalation** or lateral movement attempts within the Windows environment.

---

## 🌐 Phase 2: Network Anomalies & Port Monitoring

Every open port is a potential doorway into the system. If a port is open and an authorized service didn't open it, it is likely a backdoor.

### Linux Port Auditing
Using the `ss` (socket statistics) tool to map listening services.

**Command Executed:**
```bash
# Displaying all listening TCP/UDP ports and their associated processes
sudo ss -tulpn
```
*Critical Finding:* Discovered process `nc` (Netcat) listening on **Port 4444**. Port 4444 is the default port for Metasploit and standard reverse shells. This is a severe, high-confidence IoC indicating an active backdoor.

### Windows Port Auditing
Using PowerShell to audit the host machine to ensure the attack didn't pivot from the VM to the host.

**Command Executed:**
```powershell
# Listing listening ports and mapping them to their Process IDs (PIDs)
Get-NetTCPConnection -State Listen | Select-Object LocalPort, OwningProcess
```
*Analysis:* Mapped expected ports (135, 445) to System processes. Found high-range ports and Port 8080, which require cross-referencing with Task Manager to ensure they belong to legitimate developer tools (like local web servers) rather than external listeners.

---

## 🔗 Phase 3: Event Correlation — "The Kill Chain"

Correlation is the core skill of a SOC Analyst. A single failed login is an anomaly; a failed login followed by a successful login and a new port opening is a confirmed breach. 

By correlating the logs generated in Phase 1 and the network data from Phase 2, I constructed the following attack timeline:

### **[INCIDENT TIMELINE: CORRELATION REPORT]**

| Timestamp | Host OS | Event Type | Event ID / Process | Critical Analyst Observation |
| :--- | :--- | :--- | :--- | :--- |
| **13:53** | Windows | Failed Logon | ID 4625 | Initial unauthorized access attempt on the Windows host. |
| **18:00** | Parrot OS | Failed Auth | `sshd[1234]` | Attacker (`192.168.1.108`) pivoted to target the Linux VM's `root` account via SSH. |
| **19:01** | Parrot OS | Log Query | `journalctl` | Confirmed the brute-force footprint in the system journal. |
| **19:29** | Parrot OS | **New Listener** | Port 4444 (`nc`)| **CRITICAL:** Attacker successfully bypassed authentication and spawned a Netcat reverse shell listener. |
| **19:42** | Windows | Admin Auth | ID 4648 | Explicit credentials used on the host. Potential credential harvesting and lateral movement back to Windows. |

**Incident Conclusion:** The correlated data proves a multi-stage attack. The adversary attempted entry via Windows, pivoted to brute-force the Linux VM, successfully established a persistent backdoor (Port 4444) via Netcat, and is now attempting to use harvested credentials to move laterally back across the network.

---

## 🛡️ Phase 4: Basic Incident Response (Containment)

Upon confirming the compromise in Phase 3, immediate containment actions are required to sever the attacker's connection without destroying forensic evidence.

**The Golden Rule of Response:** Contain > Eradicate > Recover. 
*(Pro-Tip: A great analyst preserves the malware in an isolated state for reverse-engineering before deleting it.)*

### Linux Containment Actions
Terminating the rogue Netcat listener and disabling any persistent services.
```bash
# Killing the specific process ID associated with the malicious port
sudo kill -9 [PID_of_Netcat]
# If attached to a service, stopping and disabling it
sudo systemctl stop [MaliciousService]
sudo systemctl disable [MaliciousService]
```

### Windows Containment Actions
Neutralizing suspect processes identified during the port audit.
```powershell
# Forcing the termination of a compromised or rogue executable
Stop-Process -Id [OwningProcess_ID] -Force
```

---

## 🤖 Phase 5: Alerting & Monitoring Automation

Manual threat hunting is not scalable. To ensure continuous visibility, I implemented basic alerting mechanisms.

* **Linux Automation:** Drafted a shell script to pipe `grep "Failed password" /var/log/auth.log` into an `alerts.txt` file, scheduled to run hourly via `crontab -e`.
* **Windows Automation:** Utilized **Windows Task Scheduler** to execute a PowerShell script daily that parses the Security Event Log for ID 4625 and 4648, exporting anomalies to a centralized CSV for morning review.

---

## 📸 Evidence & Artifacts

*(Visual proof of the investigation, correlating directly to the incident timeline).*
 **Practical evidence** [View scan_logs.sh Script & Execution Screenshots](Evidence/)

### 1. Windows Threat Hunting (Explicit Credentials)
*Description: Querying Event ID 4648 via PowerShell to identify potential privilege escalation attempts.*
### 2. Linux Brute Force Simulation & Detection
*Description: Injecting a simulated SSH attack and subsequently querying journalctl to isolate the attacker's IP and attempt frequency.*
### 3. Network Anomaly Detection (The Smoking Gun)
*Description: Uncovering the unauthorized Netcat (`nc`) listener on Port 4444, confirming a successful system compromise.*
### 4. Cross-Platform Port Audit
*Description: Auditing the Windows host network connections to verify containment and check for lateral movement.*

---

## 🧠 Daily Reflection
Today was the tipping point between IT Administration and Cybersecurity. By learning to correlate a failed Windows login with an open Linux port, I practically applied the "Kill Chain" methodology. It became clear that individual log entries are just noise; security value is entirely derived from context and timeline correlation. Automation will be my next major focus to ensure these IoCs are flagged the second they occur, rather than hours after the breach.
```
