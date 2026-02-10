# Day 02 – Networking Fundamentals & Packet Analysis

## Overview
Day 2 transitions from the local system to the network, the primary battleground of cybersecurity. Understanding how data moves, how services are exposed, and how to intercept and analyze traffic is the prerequisite for both offensive security and defensive monitoring.
This day focused on "visibility"—the ability to see what is happening on the wire and interpret the digital signatures of various protocols.

## Objectives
* Master the OSI Model as a diagnostic and security framework.
* Verify network reachability and pathing using ICMP.
* Enumerate listening services and understand Port logic.
* Perform active network discovery and OS fingerprinting with Nmap.
* Deconstruct DNS and DHCP infrastructure protocols.
* Conduct deep packet inspection using Wireshark and tcpdump.

## The Networking Stack: Theory & Framework

### 1. The OSI Model (Layered Security)
The 7-layer OSI model was analyzed not just as theory, but as a roadmap for troubleshooting and attacks.
* **Key Concept:** Encapsulation. Data is "wrapped" in headers as it moves down the stack.

* **Security Perspective:** Attacks can happen at any layer (e.g., Layer 2 MAC spoofing, Layer 3 IP spoofing, Layer 7 Web exploits).

## Hands-On Technical Execution

### 2. Connectivity & Path Discovery
Utilized Ping and Traceroute to audit the network path.
* **Ping:** Used ICMP Echo Requests to test latency and RTT (Round Trip Time).
* **Traceroute:** Observed the TTL (Time to Live) decrementing at each hop, revealing the routers between the VM and the target.
* **Key Command:** `traceroute -n google.com` (Used -n to prevent DNS delays during path discovery).

### 3. Port Enumeration & Service Mapping
Identified entry points on the local system.
* **TCP/UDP Ports:** Understood ports as "apartment numbers" for services.
* **Tools:** Used `ss` (Socket Statistics) and `netstat`.
* **Key Command:** `sudo ss -tulpn` (Identified PIDs and process names bound to specific ports).
* **Service Dictionary:** Referenced `/etc/services` to correlate port numbers with standardized protocols.

### 4. Active Reconnaissance (Nmap)
Mastered Nmap for network auditing.

#### Techniques:
* **Ping Sweep:** `nmap -sn` to discover live hosts without alerting firewalls with a full port scan.
* **Service Versioning:** `nmap -sV` to identify exactly what software (and version) is running on an open port.
* **OS Fingerprinting:** Used `-O` and `-A` to detect the underlying Operating System through TCP/IP stack behavior analysis.

### 5. Infrastructure Protocols (DNS & DHCP)
Analyzed the "helper" protocols that maintain network order.
* **DNS:** Investigated A, AAAA, and MX records using `dig` and `nslookup`.
* **DHCP:** Observed the DORA process (Discover, Offer, Request, Acknowledge).

* **Command:** `sudo dhclient -v` (Manually triggered a DHCP lease renewal to observe the handshake).

### 6. Packet Analysis (Wireshark)
Performed live traffic sniffing and protocol deconstruction.

* **Key Filters:** `icmp`, `dns`, `tcp.port == 80`.
* **Analysis:** Captured and inspected the TCP Three-Way Handshake (SYN, SYN-ACK, ACK).
* **Troubleshooting:** Resolved VM "blank screen" issues by configuring Bridged Networking and enabling Promiscuous Mode to allow the virtual NIC to see all traffic on the segment.

## Badges & Progress Evidence
* **TryHackMe Profile:** [https://tryhackme.com/p/CyberpunkSue](https://tryhackme.com/p/CyberpunkSue)
* **Evidence:** Completion of "Intro to Networking," "Nmap," "HTTP in detail," and "DNS in Detail" modules.
* **Wireshark Trace:** Successfully captured and filtered a 4-packet ICMP exchange and an HTTP GET request.

## Security-Relevant Takeaways
* **Visibility is Security:** You cannot protect (or attack) what you cannot see. Wireshark and Nmap provide the necessary sight.
* **Version Matters:** In Hour 4, I learned that knowing a service is "Open" isn't enough; knowing it's "OpenSSH 7.2p2" tells a researcher if it's vulnerable to specific exploits.
* **The "Nesting Doll" Effect:** Seeing encapsulation in Wireshark proved how a single packet carries MAC, IP, and Port data simultaneously to reach its destination.

## Challenges Encountered & Resolutions
* **Challenge:** Wireshark showed no interfaces or zero packets.
* **Technical Root Cause:** VirtualBox NAT mode isolates the guest; permissions restricted raw socket access.
* **Resolution:** Launched Wireshark with `sudo`, switched VM adapter to Bridged Mode, and enabled Promiscuous Mode in the VirtualBox advanced settings.

## Skills Mapping (Day 02)

| Skill Area | Evidence |
| :--- | :--- |
| Network Auditing | Usage of Nmap for host discovery and versioning. |
| Traffic Analysis | Wireshark packet dissection and display filter mastery. |
| Protocol Literacy | Deep understanding of OSI layers, TCP/UDP, DNS, and DHCP. |
| Connectivity Troubleshooting | Proficient use of Ping, Traceroute, and ss. |
| VM Infrastructure | Advanced configuration of VirtualBox network adapters for sniffing. |

## Next Steps
Day 3 will transition into Network Security & Defense, focusing on:
* Firewall configurations (UFW/IPTables).
* Understanding the basics of VPNs and encrypted tunnels.
* Introductory Network Defense concepts (IDS/IPS).

**End of Day 02**
