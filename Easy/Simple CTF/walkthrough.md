# Simple CTF -- TryHackMe Writeup

**Difficulty:** Easy
**Category:** Web Exploitation / Linux Privilege Escalation

## Introduction
Simple CTF is an easy-rated TryHackMe room covering basic web exploitation and privilege escalation. It begins with reconnaissance using `nmap` to identify open ports and services, then moves into identifying the CMS in use, determining which known CVE it is vulnerable to, and exploiting that vulnerability to gain initial access. After establishing a foothold, the room walks through simple enumeration and a sudo-based privilege escalation using `vim`, answering questions along the way to reinforce each concept.

## Question 1: How many services are running under port 1000?
A scan of all ports below 1000:
```
nmap -sV -sC -p 1-1000 -T4 10.65.167.220
```
<img width="1541" height="910" alt="Screenshot_2026-05-21_16-59-41" src="https://github.com/user-attachments/assets/d6a41861-fbc9-4671-b177-30de28e85d49" />

- FTP running on port 21, specifically vsftpd 3.0.3
- HTTP running on port 80, specifically Apache httpd 2.4.18
- SSH was not found on port 22, meaning it is most likely running on port 2222 (the most common alternative)

The answer is 2 services: FTP and HTTP.

## Question 2: What is running on the higher port?
The second scan confirmed SSH is running on the alternative port 2222:
<img width="1364" height="915" alt="Screenshot_2026-05-21_17-04-53" src="https://github.com/user-attachments/assets/075f8af5-0004-494e-8170-8d8e7691dfc6" />

## Questions 3, 4, 5: CVE, vulnerability type, and password
Before identifying the CVE, I enumerated the web application with `ffuf`, since the `nmap` scan only showed the default Apache page, suggesting the rest of the application was hidden:
```
ffuf -u http://10.65.167.220/FUZZ -w /usr/share/wordlists/dirb/big.txt -e .php,.txt,.py,.html,.js -mc 200,301,302
```
<img width="1912" height="765" alt="ffuf_scan" src="https://github.com/user-attachments/assets/7c2151bc-6478-4801-b580-1e1334c7a6f5" />

- `index.html` is the default Apache page.
- `robots.txt` contained `Disallow: /openemr-5_0_1_3`, worth noting since OpenEMR is an open-source electronic health application that could carry a vulnerability.
- `simple` leads directly to the main page of the CMS Made Simple site, the actual attack vector.

Scrolling to the bottom of the CMS page revealed the version number in use:
- CMS Made Simple is an open-source, PHP-based content management system that uses MySQL, allowing website management through themes, modules, and an admin panel.

<img width="1240" height="317" alt="Screenshot 2026-05-21 at 17-34-12 Home - Pentest it" src="https://github.com/user-attachments/assets/e5cb87a2-eb08-4f46-88b9-a149b30ab24f" />

Referencing Exploit-DB for this specific version surfaced a matching exploit, a SQL injection:
<img width="1909" height="236" alt="searchsploit_scan" src="https://github.com/user-attachments/assets/1b61d9a2-95bc-433c-9519-e36c9825439b" />

### How the Exploit Works: CVE-2019-9053 (Time-Based Blind SQL Injection)
Before running the exploit, I looked up what it actually does:
<img width="542" height="338" alt="Screenshot 2026-05-21 at 17-55-01 cms made simple sql injection exploit 2 2 8 - Google Search" src="https://github.com/user-attachments/assets/687d0eb0-7585-4910-b6d4-963aca6765f9" />

CVE-2019-9053 is a time-based blind SQL injection in CMS Made Simple 2.2.8. In a blind injection, the application does not return query results directly, so the attacker cannot simply read data off the page. Instead, the exploit injects SQL that includes a conditional time delay, for example, "if the next character of the password hash is X, wait 5 seconds." By measuring how long the server takes to respond, the exploit infers each character one at a time, gradually reconstructing the administrator's username and password reset hash (and, with a wordlist, cracking the password). It is slow by nature, since each character requires its own timed request, but it reliably extracts data the application never intended to expose.

Running the exploit:
<img width="756" height="199" alt="exploit_success" src="https://github.com/user-attachments/assets/a53b8297-8f4a-471f-8da1-a55a4e9a7a07" />

This answers Questions 3 through 5:
- CVE-2019-9053
- sqli
- secret

### Common issues
A couple of issues came up while running the exploit:

- The exploit ships as Python 2 code. You can run it under Python 2, or port it to Python 3. I found it easier to update it to Python 3 syntax, which mostly came down to the print statements and ensuring strings were encoded.
- The wordlist you supply to the exploit must be UTF-8 encoded, otherwise the exploit hits a decode error and fails to recover the password.
<img width="938" height="535" alt="Screenshot_2026-05-30_18-14-54" src="https://github.com/user-attachments/assets/e96a3156-4121-4d7e-8012-08f25bdd378a" />

## Questions 6, 7, 8: Login location, user flag, and other user
The exploit output provided a username and password:
- `mitch`
- `secret`

Logging in, the user flag was found:
<img width="551" height="200" alt="Screenshot_2026-05-30_18-25-48" src="https://github.com/user-attachments/assets/bfec6ffd-052d-4364-b0a8-79dbf9772571" />

Moving out of `mitch`'s home directory revealed another home directory named `sunbath`:
<img width="883" height="382" alt="Screenshot_2026-05-30_18-28-30" src="https://github.com/user-attachments/assets/30244561-8136-48a9-8344-5133f4c3273e" />

## Questions 9 and 10: Privileged shell and root flag
### Stabilizing the shell
Before escalating, I stabilized the shell into a proper interactive TTY:
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```
<img width="782" height="90" alt="Screenshot_2026-05-30_18-34-39" src="https://github.com/user-attachments/assets/3e80918f-9e6b-4977-af42-20bd6aa2861e" />

### How the Exploit Works: Sudo on vim
With the shell cleaned up, I checked `mitch`'s sudo permissions:
<img width="862" height="94" alt="Screenshot_2026-05-30_18-37-16" src="https://github.com/user-attachments/assets/914d9526-335b-4603-814a-893d2069e4e2" />

`mitch` can run `vim` as root without a password. `vim` is a text editor, but it includes a feature for running shell commands from inside the editor. When a program with a shell-escape feature can be run as root via `sudo NOPASSWD`, that shell escape inherits root's privileges, turning the editor into a root shell. This pattern applies to any such binary, and GTFOBins documents the technique for dozens of them.

### Exploiting Sudo on vim
Create a file with `vim`:
<img width="609" height="46" alt="Screenshot_2026-05-30_18-51-41" src="https://github.com/user-attachments/assets/267755d3-3602-49fa-abc6-32804aaa102f" />

Once inside, in normal mode, run the following to spawn a root shell (since `vim` is running with root privileges):
```
:!/bin/bash
```
<img width="935" height="798" alt="Screenshot_2026-05-30_18-52-15" src="https://github.com/user-attachments/assets/e052c4c7-42f2-48fb-b232-b73ea855712c" />

With the root shell, the root flag was found:
<img width="544" height="145" alt="root" src="https://github.com/user-attachments/assets/4fec6c1c-b99c-410a-b7b8-669c2a727d0b" />

## Flag Capture
Both flags were recovered: the user flag after logging in as `mitch` with the credentials from the CVE-2019-9053 SQL injection, and the root flag after abusing `mitch`'s passwordless `sudo vim` permission to spawn a root shell.

## Lessons Learned
- **Do not assume default ports.** When SSH did not appear on port 22, checking common alternatives confirmed it was simply relocated to 2222. Scan the full range rather than trusting defaults.
- **Enumeration is the foundation.** The default Apache page hid the real attack vector. `ffuf` and a quick read of `robots.txt` exposed the CMS that everything else depended on.
- **Version numbers are gold.** Identifying the exact CMS Made Simple version is what made matching it to CVE-2019-9053 possible. A footer version string is worth scrolling for.
- **Understand the exploit before running it.** Knowing this was a time-based blind SQL injection explained both how it worked and why it was slow.
- **Mind your tooling and inputs.** The exploit shipped as Python 2 and needed porting to Python 3, and the wordlist had to be UTF-8 encoded, an overlooked encoding issue that otherwise caused a decode error and a failed run.
- **Stabilize your shell early.** Upgrading to a proper interactive TTY made enumeration and privilege escalation far smoother.
- **Check sudo -l first for privesc.** A single misconfigured sudo entry (`vim` as root, no password) was the entire path to root. Legitimate binaries with elevated rights are a classic, high-impact misconfiguration.

## Tools Used
- `nmap`
- `ffuf`
- `searchsploit`
- `vim`
