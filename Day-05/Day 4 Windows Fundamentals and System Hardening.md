# Day 4: Windows Fundamentals & System Hardening

##  Overview
Today's focus shifted from the Linux command line to the enterprise powerhouse: **Windows Fundamentals**. Understanding the Windows architecture is critical for a SOC Analyst, as the majority of corporate endpoints run on this OS. This module covered the OS architecture, the Registry, Access Control Lists (ACLs), PowerShell telemetry, Event Log forensics, and proactive system hardening.

---

##  Hour 1: Windows Architecture & Process Analysis

### Technical Summary
Windows NT uses a strict dual-mode architecture to protect the OS from applications. 
* **User Mode (Ring 3):** Applications (like browsers) run here in isolated virtual memory. They cannot access hardware directly.
* **Kernel Mode (Ring 0):** The OS core, executive services, and hardware drivers run here with full privileges. 



### Live Process Audit
Using Task Manager (configured with PID, User name, and Image Path columns), I audited my live system to identify critical "VIP" processes:
* **System (PID 4):** The NT Kernel. Verified it runs as `SYSTEM`.
* **smss.exe:** The Session Manager (first user-mode process).
* **lsass.exe:** Local Security Authority Subsystem. Verified it runs from `C:\Windows\System32`. This is a high-value target for attackers attempting credential dumping.
* **services.exe:** The Service Control Manager.

### Reflection
**Q: Why does a bad driver in Kernel Mode crash the whole system?**
**A:** Because Kernel Mode lacks memory isolation. If a driver attempts to write to protected memory or fails, the CPU triggers a "Bug Check" (Blue Screen of Death) to prevent data corruption.

---

##  Hour 2: File System Layout & Registry

### The NTFS Filesystem
Explored the hierarchical structure of the `C:\` drive. Unlike Linux, NTFS utilizes robust Access Control Lists (ACLs) and journaling. 
* **Critical Path:** `C:\Windows\System32` houses essential binaries (like `cmd.exe`) and drivers.

### The Windows Registry (The OS "Brain")
Accessed `regedit` to explore the hierarchical database that stores all OS and application configurations.



* **HKLM (HKEY_LOCAL_MACHINE):** Stores machine-wide settings (Hardware, Security, System software).
* **HKCU (HKEY_CURRENT_USER):** Stores settings specific to the currently logged-in user profile.

### Reflection
**Q: Where is the registry stored on disk?**
**A:** Machine-wide hives are stored as files in `C:\Windows\System32\Config\`. User-specific hives are stored as `NTUSER.DAT` in the respective `C:\Users\<Username>` folder.

---

##  Hour 3: Users, Groups, and Access Control (ACLs)

### Identity & Privilege
In Windows, users are identified by **SIDs (Security Identifiers)**. Permissions are typically managed via **Groups** rather than individual accounts.
* Ran `whoami /groups` to verify my security context and membership in the `BUILTIN\Administrators` group.

### NTFS Permissions Audit
Inspected the security properties of `C:\Windows\System32\drivers`. 
* **Observation:** The `Administrators` group has 'Full Control', but standard `Users` only have 'Read & Execute'. 
* **Importance:** This enforces the **Principle of Least Privilege**, preventing standard accounts from tampering with kernel-level driver files.

### Reflection
**Q: If Alice is in the "Users" group, what ACL entry gives her read access to her Documents?**
**A:** An **ACE (Access Control Entry)** within the folder's Discretionary Access Control List (DACL) that explicitly grants 'Read' permissions to her unique User SID.

---

## Hour 4: PowerShell Basics for Defenders

### Technical Reconnaissance
Transitioned from the GUI to **PowerShell**, which returns data as actionable *objects* rather than plain text. This allows for surgical precision when hunting for threats.

### Executed Cmdlets
* `Get-Process | Sort-Object CPU -Descending | Select-Object -First 10`: Enumerated top resource-consuming processes.
* `Get-Service -Name "WinDefend"`: Verified the status of Windows Defender.
* `Get-NetTCPConnection -State Established`: Audited active outbound and inbound network connections.
* `Get-Service | Where-Object Status -eq "Running"`: Used the pipe (`|`) to filter for active services.

---

## Hour 5: Event Logs and Forensics

### The "Flight Recorder" of Windows
Explored the Windows Event Logging system, focusing on the three main logs: **System, Security, and Application**.

### Target Event IDs Tracked:
* **Event ID 4624:** Successful Logon.
* **Event ID 4625:** Failed Logon (Indicator of brute-force attacks).
* **Event ID 4720:** User Account Creation (Indicator of unauthorized persistence).

### Hands-On Investigation
* Utilized **Event Viewer (`eventvwr.msc`)** to filter the Security Log specifically for Event ID 4624, allowing me to audit recent successful authentications on my machine.
* Executed `Get-WinEvent -LogName Security -MaxEvents 5` via PowerShell for rapid, command-line forensic retrieval.

---

## Hour 6: Attack Surface Reduction & Hardening

### Proactive Defense
Concluded the day by applying hardening techniques to minimize the system's attack surface.

### Applied Security Controls:
1. **Service Hardening:** Audited `services.msc` and successfully changed the **Remote Registry** service startup type to `Disabled`. This closes a potential remote configuration tampering vector.
2. **UAC Enforcement:** Verified that **User Account Control (UAC)** is active, ensuring privilege separation is enforced even for Admin accounts.
3. **Patch Management:** Audited Windows Update to ensure the system is protected against known zero-day vulnerabilities.

---

## Badges & Progress Evidence
* **System Vitals Audit:** Successfully verified core system processes, mapping them to their correct Ring 0/Ring 3 contexts.
* **Forensic Capability:** Navigated Event Logs to identify successful authentication events using Windows Event IDs.
* **Access Control & Hardening:** Enforced service restrictions and audited NTFS permissions.

### 🖼️ Technical Evidence Vault
* **Practical evidence:** [View scan_logs.sh Script & Execution Screenshots](Evidence/)
* **[Process Config Setup]:** Configuring Task Manager for security auditing.
* **[Critical Process Audit]:** Verification of `lsass.exe` and `services.exe` legitimacy.
* **[Registry Mapping]:** Navigating the Windows Registry database structure.
* **[Registry Core Config]:** Inspecting the HKLM hive for system-wide software configurations.
* **[Log Analysis]:** Investigating the Security Event Log for Event ID 4624 (Successful Logons).
* **[Hardening Implementation]:** Disabling the Remote Registry service to reduce the system's attack surface.
