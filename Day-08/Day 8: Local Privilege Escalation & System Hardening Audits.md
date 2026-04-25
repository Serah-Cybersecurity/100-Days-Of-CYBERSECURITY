# Day 8: Local Privilege Escalation & System Hardening Audits

**Objective:** Transition from external network reconnaissance to internal system auditing. This module focuses on identifying misconfigurations within the Linux operating system, understanding the mechanics of permission inheritance, and validating "attack paths" that allow a low-level user to achieve full **root** (Administrative) compromise.

**Mindset Shift:** Moving from "How do I get in?" to "How do I climb?" Today’s focus is acting as a **System Auditor** to identify architectural flaws—such as insecure SUID bits, sudo misconfigurations, and writable system files—and documenting the remediation steps required to harden the environment.

**Lab Environment:**
* **Hardware:** Lenovo ThinkPad X390 Yoga.
* **OS:** Linux Mint 22 (Wilma) / Parrot Security environment.
* **Tools:** LinPEAS (Privilege Escalation Awesome Script), GTFOBins, Terminal-based enumeration (find, grep, ls, sudo).

---

## 🕒 Hour 1: The Hierarchy of Permissions & Mental Models
The objective was to establish a baseline of the current user's environment and understand the two primary directions of movement within a compromised system.

* **Vertical Privilege Escalation:** Moving from a standard user (e.g., `cs`) to a superuser (`root`).
* **Horizontal Privilege Escalation:** Moving from one standard user to another to access different datasets or SSH keys.
* **Initial Interrogation:** Used `whoami`, `id`, and `hostname` to confirm the starting point and group memberships.

---

## 🕒 Hour 2: Automated System Auditing (LinPEAS)
Rather than checking thousands of files manually, I utilized **LinPEAS**, the industry-standard automated enumerator, to identify "low-hanging fruit" and high-risk misconfigurations.

* **Command Executed:** `./linpeas.sh > linpeas_results.txt`
* **Analysis of Output:**
    * **Kernel Context:** Identified the system as running Kernel `6.8.0`, which is modern and largely patched against famous automated root exploits (e.g., DirtyPipe).
    * **PATH Hijacking:** Discovered that `/home/cs/.local/bin` is user-writable and included in the system PATH, presenting a risk for binary hijacking.
    * **Hardening Status:** Found that **AppArmor** is active but "unconfined," indicating the security framework is present but not yet enforcing strict profiles on all applications.

---

## 🕒 Hour 3: The "Big Three" Local Privilege Paths
This phase involved manual verification of the "Red/Yellow" flags raised by the automated scan. I focused on the three most common ways a Linux system is compromised from the inside.

### 1. SUID Binary Hunt
* **Action:** Searched for files with the "Set User ID" bit, which allows a program to run with root authority.
* **Discovery:** Identified `/usr/bin/mount` and `/opt/lampp/bin/suexec` as SUID binaries. These are now high-priority audit targets.

### 2. Sudo Privileges
* **Action:** Executed `sudo -l` to map the current user's "superpowers."
* **Discovery:** The user `cs` has `(ALL : ALL) ALL` permissions. While necessary for an admin, this represents a "Total Compromise" risk if the account is hijacked.

### 3. Critical File Permissions
* **Action:** Audited the `/etc/` directory for world-writable files.
* **Discovery:** Confirmed that sensitive files like `/etc/shadow` (password hashes) are correctly locked down with `-rw-r-----` permissions, accessible only by root and the shadow group.

---

## 🕒 Hour 4: Enterprise Lateral Movement (AD Concepts)
Privilege escalation in a corporate environment often moves from a single Linux machine into the **Active Directory (AD)** domain.

* **Lateral Movement:** Moving between workstations to find cached credentials.
* **BloodHound Mapping:** Conceptually mapped how graph theory is used to find the shortest path from a "Standard User" to a "Domain Admin."
* **Defense (Tiered Administration):** Documented the requirement that high-privilege admins must never log into lower-tier workstations to prevent credential harvesting.



---

## 🕒 Hour 5: Technical Validation & Safe Proofing (GTFOBins)
Correlation is the mark of a professional. I cross-referenced my findings with the **GTFOBins** database to prove the risk without executing destructive exploits.

* **Validation Case Study:** The `mount` binary.
* **The Chain of Evidence:**
    1.  **LinPEAS** flagged it as SUID.
    2.  **GTFOBins** confirmed it can be abused for "Arbitrary File Read."
    3.  **Manual Check** (`ls -l`) confirmed the `s` bit is live on the system.
* **Conclusion:** The vulnerability is **Validated** as a True Positive.

---

## 🕒 Hour 6: Mitigation & Hardening Strategy
A security professional must provide a clear path to recovery. I drafted the following remediation strategy:

1.  **Remove Unnecessary SUID Bits:** Any binary not requiring elevated privileges for standard users should have the SUID bit stripped (`chmod u-s`).
2.  **Principle of Least Privilege (PoLP):** Transition from `(ALL : ALL) ALL` sudo access to specific, command-limited aliases in the `/etc/sudoers` file.
3.  **Persistence Monitoring:** Regularly audit the `/etc/cron*` directories for unauthorized scheduled tasks that could be used for maintaining root access.

---

## 📸 Evidence & Artifacts
*(Visual proof of the investigation, correlating directly to the incident timeline).*
 **Practical evidence** [](Evidence/)

| Artifact Name | Description |
| :--- | :--- |
| `Manual_SUID_Audit.png` | Documentation of the `find` command identifying elevated binaries. |
| `Shadow_File_Permissions.png` | Verification that the system's credential store is properly hardened against unauthorized reads. |
| `Sudo_Privilege_Verification.png` | Screenshot of the `sudo -l` output defining the user's administrative reach. |
| `Cron_Directory_Inspection.png` | Evidence of a persistence audit across system-wide scheduled task folders. |
| `Validation_Source_GTFOBins.png` | Research evidence showing the known exploit path for the `mount` binary. |
| `Local_Verification_Mount_Binary.png` | The "Smoking Gun": Proving the metadata on the local system matches the known vulnerability. |
| `System_Hardening_AppArmor.png` | Documentation of the current OS hardening status and kernel version. |

---

## 🧠 Daily Reflection
Today shifted the focus from "finding a bug" to "analyzing an architecture." By running LinPEAS and then manually verifying the results via `ls`, `grep`, and **GTFOBins**, I learned that automated tools only provide the *questions*—the security professional must provide the *answers*. Documenting the "Successes" (like the locked-down `/etc/shadow`) is just as important as finding the "Flaws." My focus is no longer on hacking; it is on **Forensic Auditing and Verifiable Remediation.**

Great work on completing Day 8. You've built a folder of evidence that would impress any hiring manager. Are you ready to push this to your repo, or should we refine that LinkedIn caption one last time?
