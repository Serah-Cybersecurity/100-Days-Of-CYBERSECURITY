# Day 03: Linux Fundamentals & Security Automation

## 🎯 Overview
Today, I moved completely away from the GUI and spent over 6 hours navigating the Parrot OS terminal. I learned that Linux isn't just an operating system; it's a hierarchical security framework. My objective was to understand access control, service management, and how to automate forensic log analysis. 

## 🧠 Key Concepts Mastered

### 1. The Filesystem Hierarchy
Linux operates on the philosophy that "Everything is a file." I navigated critical security paths to understand where a system is most vulnerable and where it stores its evidence.
* **`/etc`**: The nerve center containing system configurations, including sensitive files like `passwd` and `shadow`.
* **`/var/log`**: The primary repository for system events and forensic evidence.



### 2. Access Control & Permissions
I applied the **Principle of Least Privilege** by mastering `chmod` and `chown`. Understanding how to lock down sensitive files using octal permissions (like `chmod 700`) is the first line of defense against Privilege Escalation.



### 3. Service Management
Learned how to use `systemctl` to manage system daemons, specifically starting and checking the status of the SSH service to enable secure connections (and test my defenses).

---

## 🤖 The Project: SOC Log Scanner Automation
I built a Bash script designed for a Security Operations Center (SOC) to automatically detect unauthorized brute-force access attempts. 

**The Script (`scan_logs.sh`):**
```bash
#!/bin/bash
# Universal Log Scanner v3.0

echo "=========================================="
echo "    🔍 SOC SECURITY SCAN INITIALIZED     "
echo "    TIME: $(date)                       "
echo "=========================================="

echo "[*] Searching for failed login attempts..."
echo "------------------------------------------"

# Queries the systemd journal for failed SSH attempts
sudo journalctl -u ssh | grep -i "failed" | tail -n 5
```
## 🕵️ Red Team Simulation & Technical Forensics

To test the efficacy of my detection tool, I had to generate "attack" telemetry. I simulated a brute-force attack by targeting the local SSH daemon.

* **Command:** `ssh fakeuser@localhost`
* **Method:** Intentionally failed the password prompt multiple times to trigger security alerts in the system journal.
* **Result:** My script successfully parsed the binary system logs, identifying the exact timestamps and unauthorized usernames associated with the failed attempts.



---

## 🛠️ Troubleshooting & Technical Pivots (The Real Work)

The most valuable part of today wasn't the successful run; it was the process of fixing the failures. Documentation of these pivots proves technical adaptability:

### 1. Service Failure: "Connection Refused"
* **The Issue:** My initial attack simulation failed because Port 22 was closed.
* **The Fix:** Diagnosed that the SSH daemon was inactive. I used `sudo systemctl start ssh` to enable the listener and verified it with `systemctl status ssh`.

### 2. Pathing & Execution Errors
* **The Issue:** Encountered "No such directory" and "Permission denied" errors. 
* **The Fix:** Used `pwd` (Print Working Directory) to verify my location and `ls -l` to audit file permissions. I then applied `chmod +x` to modify the file mode bits, granting the system permission to execute my script.



### 3. Terminal Input Hangs
* **The Issue:** Standard text editors and the `cat` command hung my terminal during script creation due to environment sync issues.
* **The Fix:** I pivoted to using the `echo` command with output redirection (`>`) and append (`>>`) operators. This allowed me to construct the script line-by-line directly into the file without an interactive editor.

### 4. Syntax Debugging (The "One Character" Rule)
* **The Issue:** A minor typo (`jounalctl` instead of `journalctl`) broke the entire automation. 
* **The Fix:** I performed a line-by-line audit of my script code. This served as a critical lesson: in cybersecurity, Linux is unforgiving with syntax, and one missing character can be the difference between a working defense tool and a total system failure.

echo "------------------------------------------"
echo "[+] Scan Complete."
echo "=========================================="
