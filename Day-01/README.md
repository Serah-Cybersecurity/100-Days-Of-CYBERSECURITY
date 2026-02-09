# Day 01 – Environment Setup & Linux Fundamentals

## Objective

The objective of Day 1 was to establish a functional cybersecurity lab environment and begin developing foundational Linux terminal skills required for security operations, incident response, and threat analysis roles.

Linux is a core operating system in cybersecurity. Comfort with the command line is essential for log analysis, system investigation, and security tooling.

---

## Lab Environment

**Host Operating System**
- Windows

**Virtualization**
- Virtual Machine running on Windows host

**Guest Operating System**
- Parrot Security OS

**Reason for Parrot OS**
Parrot OS is designed for cybersecurity professionals and includes pre-installed tools commonly used in defensive and offensive security workflows. It provides a realistic environment aligned with industry practices.

---

## Activities Performed

### Linux Terminal Orientation
- Opened and interacted with the Linux terminal
- Understood the role of the shell as an interface to the operating system

### Filesystem Navigation
Practiced navigating the Linux filesystem using core commands:
- `pwd` to identify current working directory
- `ls` to list directory contents
- `cd` to move between directories

Understood how Linux uses a hierarchical directory structure starting from the root (`/`), which differs from Windows drive-based navigation.

### File and Directory Management
- Created files using `touch`
- Created directories using `mkdir`
- Edited files using terminal-based text editors
- Removed files and directories safely

This introduced the concept of directly manipulating system resources without a graphical interface.

### Permissions Awareness (Introduction)
- Learned that Linux uses permission sets (read, write, execute)
- Understood that improper permissions can expose sensitive files or enable unauthorized actions
- Recognized permissions as a critical security control

---

## Security-Relevant Takeaways

- Linux terminal fluency is non-negotiable for cybersecurity roles
- Many security tools and logs are accessed exclusively via command line
- Misconfigured files and permissions are common security weaknesses
- Understanding system structure improves incident response speed and accuracy
- Security professionals must be comfortable operating in minimal or no-GUI environments

---

## Challenges Encountered

- Initial difficulty remembering command syntax and navigation patterns
- Confusion between relative and absolute paths

**Resolution**
Repeated hands-on practice by intentionally navigating incorrectly, correcting mistakes, and rebuilding directory structures to reinforce understanding.

---

## Evidence of Hands-On Work

- Successfully navigated directory structures
- Created and edited files from the terminal
- Demonstrated basic command-line confidence within the Parrot OS environment

(Screenshots and additional artifacts may be added as the lab environment evolves.)

---

## Next Steps

Day 2 will focus on networking fundamentals, including:
- How devices communicate
- IP addressing and protocols
- Basic traffic inspection and reconnaissance from a defensive perspective
