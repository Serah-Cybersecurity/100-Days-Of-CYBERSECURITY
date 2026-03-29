# Day 5: System Visibility & Log Monitoring for Cybersecurity

## 🎯 Objective
Transition from observing a system as a user to auditing it as a **Security Operations Center (SOC) Analyst**. This module covers process tracking, log analysis, cross-platform security event mapping (Windows/Linux), and automation scripting.

## 🛠️ Lab Environment
* **VirtualBox:** Hosting Parrot OS (Debian-based)
* **Windows Host:** PowerShell auditing
* **Network:** TryHackMe Lab Network

---

## Phase 1: Live System Monitoring (Processes & Services)

Understanding the "heartbeat" of a system is the first step in identifying anomalies. A **Process** is an active task executing code, while a **Service** is a background worker (daemon).

### Linux Visibility
Utilized `top`/`htop` for interactive monitoring and `ps aux` for snapshots.

**Command Executed:**
```bash
# Sorting the top 11 processes by CPU usage to identify resource-heavy anomalies
ps aux --sort=-%cpu | head -n 11
```

### Windows Visibility
Prioritizing PowerShell for speed and scriptability over the GUI Task Manager.

**Command Executed:**
```powershell
# Identifying the top 10 most CPU-intensive processes
Get-Process | Sort-Object CPU -Descending | Select-Object -First 10
```

> **Security Insight:** Attackers often mask malicious processes by naming them similarly to legitimate ones (e.g., `svch0st.exe` instead of `svchost.exe`).

---

## Phase 2: Locating the Evidence (Log Fundamentals)

Logs are the "security camera footage" of an OS. Without them, there is no proof of an incident.

### Core Log Locations

| OS Environment | Primary Log Path / Tool | Purpose |
| :--- | :--- | :--- |
| **Linux (Debian/Ubuntu)** | `/var/log/auth.log` | Authentication (Logins, sudo usage) |
| **Linux (RHEL/CentOS)** | `/var/log/secure` | Authentication for Red Hat systems |
| **Linux (systemd)** | `journalctl -u ssh` | Binary journal querying |
| **Windows** | Event Viewer -> Security | Audit Success and Failure events |

**Command Executed (Linux):**
```bash
# Querying the systemd journal for all failed authentication attempts
sudo journalctl | grep -i "failed"
```

---

## Phase 3: Sifting Through the Noise (Filtering & Searching)

### Linux Text Filtering (`grep`)
```bash
# Searching specifically for failed password attempts
grep -i "failed" /var/log/auth.log

# Alternative using journalctl
journalctl _AUDIT_TYPE_NAME=USER_AUTH | grep -i "failed"
```

### Windows Object Filtering (`Where-Object`)
Focusing on **Event ID 4625** (Failed Logon).

```powershell
# Querying the Security log for the 5 most recent failed logins
Get-EventLog -LogName Security -InstanceId 4625 -Newest 5
```

**Logon Type Context:**
* **Type 2:** Interactive (Physical access)
* **Type 3:** Network (Shared folders)
* **Type 10:** Remote Desktop (RDP) — *High risk for external attacks.*

---

## Phase 4: Security Automation Scripts

### Linux Bash Script (`check_logs.sh`)
```bash
#!/bin/bash
echo "--- DAILY SECURITY REPORT ---"

if [ -f /var/log/auth.log ]; then
    grep -i "failed" /var/log/auth.log | tail -n 10
elif [ -f /var/log/secure ]; then
    grep -i "failed" /var/log/secure | tail -n 10
else
    echo "No standard auth log file found. Querying journalctl..."
    journalctl | grep -i "failed" | tail -n 10
fi
```

### Windows PowerShell Script (`CheckLogs.ps1`)
```powershell
Write-Host "--- CRITICAL SECURITY EVENTS (LAST 24H) ---" -ForegroundColor Cyan
Get-EventLog -LogName Security -InstanceId 4624, 4625 -Newest 10 | Select-Object TimeGenerated, InstanceId, Message | Format-Table
```

---

## Phase 5: Cross-Platform Visibility (Master Translation)

| Security Event | Linux Methodology | Windows Event ID |
| :--- | :--- | :--- |
| **Active Processes** | `ps aux` / `htop` | `Get-Process` |
| **Background Services**| `systemctl` | `Get-Service` |
| **Successful Login** | `/var/log/auth.log` | **4624** |
| **Failed Login** | Search "Failed password" | **4625** |
| **Privilege Escalation**| `sudo` / `su` | **4672** |
| **New User Created** | `useradd` | **4720** |
| **Clearing Logs** | `history -c` / `rm` | **1102** (Critical Alert) |

---

## 🔧 Technical Sidebar: VM Troubleshooting
**Issue:** Host-to-guest keyboard mapping desynchronization in VirtualBox.
1. **Session Fix:** `setxkbmap us`
2. **Persistent Fix:** Navigated to *System -> Preferences -> Hardware -> Keyboard -> Layouts*.
3. **Driver Sync:** Re-mounted VirtualBox Guest Additions.

---

## 📸 Evidence & Artifacts
1. **Process Monitoring:** `![Process Monitoring](assets/process_mon.png)`
2. **Log Querying:** `![Log Query](assets/log_query.png)`
3. **Automation Result:** `![Script Execution](assets/script_result.png)`

---

## 🧠 Summary & Mindset Shift
Today marked a shift from observing systems to auditing them. Logs are not just text; they are a chronological narrative. Building cross-platform visibility and automating log parsing are the foundational skills required for a SOC Analyst role. **You cannot secure what you cannot see.**
```
