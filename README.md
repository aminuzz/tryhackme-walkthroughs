# TryHackMe Walkthroughs

A collection of detailed walkthroughs for TryHackMe rooms I have completed, organized by the primary skill and attack vector each room focuses on. Every writeup documents the full process from start to finish: enumeration, the foothold and how the exploit works, privilege escalation, flag capture, and the tools used.

## Topics

- [Web Exploitation](#web-exploitation)
- [CVE and Remote Code Execution](#cve-and-remote-code-execution)
- [Steganography](#steganography)
- [OSINT](#osint)
- [Network and Traffic Analysis](#network-and-traffic-analysis)
- [Enumeration and Linux Fundamentals](#enumeration-and-linux-fundamentals)
- [Password and Hash Cracking](#password-and-hash-cracking)

---

## Web Exploitation
Rooms where the foothold comes through a web application: file-upload bypasses, broken access control, CMS flaws, and enumeration-driven entry points.

| Room | Difficulty | Key Techniques |
|------|------------|----------------|
| [grep](Web-Exploitation/grep/walkthrough.md) | Easy | Source-code review, commit-history secrets, magic-byte upload bypass, breach lookup |
| [Root Me](Web-Exploitation/Root%20Me/walkthrough.md) | Easy | Arbitrary file upload, reverse shell, sudo privilege escalation |
| [Pickle Rick](Web-Exploitation/Pickle%20Rick/walkthrough.md) | Easy | Source-code recon, restricted command panel, sudo misconfiguration |
| [Startup](Web-Exploitation/Startup/walkthrough.md) | Easy | Anonymous FTP write to web root, pcap credential harvest, writable root-cron script |
| [ToolsRus](Web-Exploitation/ToolsRus/walkthrough.md) | Easy | Directory enumeration, Nikto, Tomcat Manager exploitation |
| [Gaming Server](Web-Exploitation/Gaming%20Server/walkthrough.md) | Easy | Web enumeration, SSH key cracking, LXD group privilege escalation |
| [Lazy Admin](Web-Exploitation/Lazy%20Admin/walkthrough.md) | Easy | CMS exploitation, reverse shell, sudo misconfiguration |
| [Easy Peasy](Web-Exploitation/Easy%20Peasy/walkthrough.md) | Easy | Web enumeration, encoding, steganography, cron-based privilege escalation |
| [Neighbor](Web-Exploitation/Neighbor/walkthrough.md) | Easy | Broken access control via parameter manipulation |
| [TakeOver](Web-Exploitation/TakeOver/walkthrough.md) | Easy | Subdomain enumeration and discovery |

## CVE and Remote Code Execution
Rooms built around a specific named vulnerability leading to remote code execution.

| Room | Difficulty | Key Techniques |
|------|------------|----------------|
| [Source](CVE-and-RCE/Source/walkthrough.md) | Easy | Webmin 1.890 backdoor, CVE-2019-15107, unauthenticated RCE as root |
| [Ignite](CVE-and-RCE/Ignite/walkthrough.md) | Easy | Fuel CMS, CVE-2018-16763, remote code execution |
| [Vulnerability Capstone](CVE-and-RCE/Vulnerability%20Capstone/walkthrough.md) | Easy | CVE research, Fuel CMS exploitation |
| [Agent T](CVE-and-RCE/Agent%20T/walkthrough.md) | Easy | PHP 8.1.0-dev User-Agentt backdoor, remote code execution |
| [Simple CTF](CVE-and-RCE/Simple%20CTF/walkthrough.md) | Easy | CMS Made Simple SQL injection, CVE-2019-9053, sudo vim privilege escalation |

## Steganography
Rooms where data hidden inside files is a core step in the attack chain.

| Room | Difficulty | Key Techniques |
|------|------------|----------------|
| [UA High School](Steganography/UA%20High%20School/walkthrough.md) | Easy | Recursive enumeration, command-injection web shell, file-signature repair, steghide, eval sudo-script privilege escalation |
| [Madness](Steganography/Madness/walkthrough.md) | Easy | File-signature repair, steghide, GNU Screen 4.5.0 SUID privilege escalation |
| [Agent Sudo](Steganography/Agent%20Sudo/walkthrough.md) | Easy | FTP enumeration, steghide, sudo bypass CVE-2019-14287 |
| [Brooklyn Nine Nine](Steganography/Brooklyn%20Nine%20Nine/walkthrough.md) | Easy | Anonymous FTP, steghide, Hydra SSH brute force, sudo privilege escalation |

## OSINT
Rooms solved by gathering and correlating open-source intelligence.

| Room | Difficulty | Key Techniques |
|------|------------|----------------|
| [OhSINT](OSINT/OhSINT/walkthrough.md) | Easy | EXIF metadata, cross-platform pivoting, WiGLE BSSID lookup |
| [Fowsniff CTF](OSINT/Fowsniff%20CTF/walkthrough.md) | Easy | OSINT credential discovery, POP3 mail, MOTD privilege escalation |

## Network and Traffic Analysis
Rooms centered on packet capture analysis and network scanning.

| Room | Difficulty | Key Techniques |
|------|------------|----------------|
| [Smag Grotto](Network-and-Traffic-Analysis/Smag%20Grotto/walkthrough.md) | Easy | Packet-capture credential harvest, writable cron-script privilege escalation |
| [Intermediate Nmap](Network-and-Traffic-Analysis/Intermediate%20Nmap/walkthrough.md) | Easy | Nmap methodology and service enumeration |

## Enumeration and Linux Fundamentals
Enumeration-heavy rooms and drills in core Linux skills.

| Room | Difficulty | Key Techniques |
|------|------------|----------------|
| [Basic Pentesting](Enumeration-and-Linux-Fundamentals/Basic%20Pentesting/walkthrough.md) | Easy | Service enumeration, SMB, SSH private-key cracking with ssh2john |
| [Bounty Hunter](Enumeration-and-Linux-Fundamentals/Bounty%20Hunter/walkthrough.md) | Easy | FTP enumeration, login bypass, sudo python privilege escalation |
| [Ninja Skills](Enumeration-and-Linux-Fundamentals/Ninja%20Skills/walkthrough.md) | Easy | Linux file-system enumeration with find, grep, and hashing |

## Password and Hash Cracking
Rooms focused on identifying and cracking hashes.

| Room | Difficulty | Key Techniques |
|------|------------|----------------|
| [Crack the Hash](Password-and-Hash-Cracking/Crack%20the%20Hash/walkthrough.md) | Easy | Hash identification, wordlist attacks with hashcat and John the Ripper |

---

Each room folder contains its full `walkthrough.md`. Writeups are my own; where a room required a public exploit written by someone else, that author is credited within the writeup.
