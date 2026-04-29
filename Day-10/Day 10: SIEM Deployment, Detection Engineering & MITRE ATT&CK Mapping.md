# Day 11: Advanced Digital Forensics, Memory Analysis & Network Triage

### 🧠 Mindset Shift: From Live Execution to Post-Mortem Auditing
Today's transition moves from the perspective of an active network operator to that of a **Forensic Investigator**. The mindset shifts from "How do I secure this live system?" to "How do I extract the truth from a frozen state?" This requires a methodological shift towards absolute evidence preservation, understanding how volatile memory (RAM) holds the absolute truth of system intent, and utilizing isolated network sandboxes to safely triage malicious artifacts. 

---

## ⏱️ Hour-by-Hour Technical Execution

**Lab Environment:** OS: Kali OS (VM) | Host: LINUX MINT (HP ELITEBOOK)
**Core Tools:** Volatility 3, Wireshark, `nslookup`, `ping`, CLI network utilities

### Hour 1: Framework Initialization & Dependency Validation
* **Goal:** Initialize the Volatility 3 framework and ensure Windows symbol tables are properly loaded for forensic parsing.
* **Action:** ```bash
  python3 vol.py -h
  ```
* **Discovery:** Confirmed the framework's readiness and validated that core Windows plugins (like `windows.info` and `windows.pslist`) were actively recognized by the system.

### Hour 2: Path Configuration & Evidence Integrity Check
* **Goal:** Resolve `FileNotFoundError` exceptions and establish a verifiable chain of custody for the raw memory image.
* **Action:** ```bash
  cd volatility3
  ls -la | grep memdump.raw
  ```
* **Discovery:** Corrected the terminal working directory path. Verified the physical presence and integrity of the `memdump.raw` file before initiating any parsing algorithms.

### Hour 3: OS Profile Identification & System Baselining
* **Goal:** Extract the target OS architectural profile to establish a baseline for memory translation.
* **Action:** ```bash
  python3 vol.py -f memdump.raw windows.info
  ```
* **Discovery:** Successfully identified the kernel architecture as Windows 10 x64. This provided the necessary Kernel Virtual Address markers required to translate raw binary into human-readable processes.

### Hour 4: Process Tree Enumeration & Behavioral Triage
* **Goal:** Reconstruct the active process hierarchy to identify orphaned or anomalous software executions.
* **Action:** ```bash
  python3 vol.py -f memdump.raw windows.pslist
  ```
* **Discovery:** Mapped parent-child process relationships. Transitioned into deeper behavioral checks looking for unlinked processes that might indicate user-mode rootkits or stealth execution.

### Hour 5: Socket Enumeration & Command Line Retrieval
* **Goal:** Link internal system processes to external network activity and extract direct user intent.
* **Action:** ```bash
  python3 vol.py -f memdump.raw windows.netstat
  # Followed by CMD history extraction plugins
  ```
* **Discovery:** Identified active IP addresses and listening ports at the moment of capture. Successfully recovered command-line history, revealing exactly what commands were typed prior to the system snapshot.

### Hour 6: Code Injection Scanning & Network Laboratory Triage
* **Goal:** Detect hidden shellcode injected into legitimate processes, followed by validating the isolated Host-Only network capture environment.
* **Action:** ```bash
  python3 vol.py -f memdump.raw windows.malfind
  ping -c 4 192.168.56.1
  nslookup forensics.lab
  ```
* **Discovery:** Conducted sweeps for DLL injection. In the network lab, successfully correlated "Network Unreachable" OS errors with live Wireshark captures, forcing ICMP and UDP (Port 53) traffic generation via the internal gateway (`eth0` interface) to prove lab isolation.

---

## 📊 The "Big Three" Technical Analysis

| Concept | Description | Technical Forensic Application |
| :--- | :--- | :--- |
| **Volatile Data Extraction** | Parsing raw RAM dumps to rebuild system state. | Utilizing `windows.pslist` vs `windows.malfind` to distinguish between legitimate running software and hidden/injected executable memory pages. |
| **Network Triage in Isolation** | Capturing packets in a Host-Only sandbox. | Verifying `eth0` bindings via `ip addr`. Filtering for `udp.port == 53` (DNS) and `icmp` to capture processes attempting to beacon out from an unreachable environment. |
| **Compliance: Policy vs. Standard** | Differentiating rules governing forensic evidence handling. | **Policy** (General mandatory rule): "All forensic memory captures must be analyzed in a strictly isolated virtual environment." <br> **Standard** (Specific metric-based rule): "All raw images must be verified with a SHA-256 hash before and after Volatility parsing to ensure zero byte manipulation." |

---

## 📁 Evidence & Artifacts Gallery

| Artifact Name | Description | Technical Observation |
| :--- | :--- | :--- |
| **Day 11 - Hour 1 - Framework Initialization** | Volatility 3 CLI help menu output. | Proves framework readiness and successful Python dependency execution. |
| **Day 11 - Hour 2 - Evidence Path Validation** | Terminal output showing `memdump.raw` in the active directory. | Documents resolution of directory pathing errors and confirms evidence staging. |
| **Day 11 - Hour 3 - OS Profile Identification** | Output of the `windows.info` plugin. | Identifies the exact OS build (Win10 x64), providing the crucial foundational layer for memory translation. |
| **Day 11 - Hour 4 - Active Memory Process List** | Output of `windows.pslist`. | Reveals the hierarchy of running processes, serving as the baseline for anomalous behavior detection. |
| **Day 11 - Hour 5 - Command Line History Retrieval** | Recovered terminal commands from RAM. | Provides the "smoking gun" of user or attacker intent immediately prior to the memory capture. |
| **Day 11 - Hour 6 - Malicious Code Injection Scan** | Output of `windows.malfind`. | Scans for memory segments with `PAGE_EXECUTE_READWRITE` permissions indicating injected shellcode. |
| **Day 11 - Hour 6 - Protocol Filtering & DNS Triage** | Wireshark capture filtering for DNS/ICMP. | Confirms network isolation while successfully capturing the target's failed outbound beaconing attempts. |

---

## 🛡️ Hardening & Mitigation Strategy

Based on the forensic artifacts recovered today, the following system hardening steps are recommended:

1.  **Implement EDR Memory Scanning:** Deploy Endpoint Detection and Response (EDR) solutions capable of active heuristic memory scanning to detect `PAGE_EXECUTE_READWRITE` anomalies (like those found via `malfind`) before they execute.
2.  **Strict DNS Egress Filtering:** Restrict outbound UDP Port 53 traffic exclusively to authorized internal corporate DNS servers. This prevents malicious processes from using rogue DNS lookups for command-and-control (C2) beaconing or data exfiltration.
3.  **Harden PowerShell & CMD Execution:** Implement strict logging for all command-line interfaces (Event ID 4688 with command-line auditing enabled) to ensure that the history we recovered forensically is also securely pushed to a centralized SIEM in real-time.
4.  **Network Segregation:** Ensure high-risk or potentially compromised machines can be dynamically shifted into strict Host-Only VLANs (similar to our forensic sandbox setup) to prevent lateral movement during an active incident.

---

## 📝 Daily Reflection (P.A.R. Method)

**Problem:** Navigating path configuration errors within the Volatility framework and failing to capture DNS traffic in Wireshark due to strict OS-level Host-Only network isolation ("Network Unreachable" errors). 

**Action:** Addressed the memory framework issue by systematically verifying directory mapping and evidence integrity. Solved the network visibility issue by utilizing internal gateway pings (`192.168.56.1`) to force the generation of Layer 3 (ICMP) and Layer 4 (UDP) traffic, synchronizing live Wireshark captures with terminal execution.

**Result:** Successfully extracted deep behavioral artifacts—including command-line history and process trees—from raw memory. Furthermore, mastered the technique of capturing targeted protocol traffic within an air-gapped forensic environment, proving a high-level understanding of both Forensic Auditing and Verifiable Remediation workflows.
