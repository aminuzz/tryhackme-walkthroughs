# ToolsRus -- TryHackMe Writeup

**Difficulty:** Easy  
**Category:** Web Enumeration / Tomcat Exploitation

## Introduction
ToolsRus is an easy-rated TryHackMe room designed to practice a toolkit of common enumeration utilities. It walks through directory discovery, HTTP basic-authentication brute-forcing, service fingerprinting, and a Nikto scan, before finishing with a Metasploit exploit against an Apache Tomcat Manager to land a root shell. The room is structured as a series of questions, each reinforcing a specific tool.

## Enumeration
### Nmap Scan
The first step was an `nmap` scan to map the open ports and services:

<!-- SCREENSHOT: nmap scan output -->

### Directory Discovery
Fuzzing for directories revealed a directory beginning with "g":

<!-- SCREENSHOT: directory fuzzing results -->

- **Answer (directory starting with "g"):** `guidelines`

Inside that directory, a name was disclosed:

<!-- SCREENSHOT: guidelines directory revealing a name -->

- **Answer (name found):** `bob`

Continuing enumeration, a directory protected by HTTP basic authentication was found:

<!-- SCREENSHOT: directory requiring basic authentication -->

- **Answer (directory with basic auth):** `protected`

### Brute-Forcing Basic Authentication
The `protected` directory prompts for a login, so `hydra` was used to brute-force `bob`'s password against the HTTP basic-auth endpoint:

<!-- SCREENSHOT: hydra brute-forcing the basic-auth login -->

- **Answer (bob's password):** `bubbles`

### Service Fingerprinting
A service-version scan identified the software running on the higher port:

- **Answer (software and version):** Apache Tomcat/7.0.88, a Java-based web server and servlet container used to run Java web applications.

## Exploitation
### Nikto Scan
Using the recovered credentials (`bob:bubbles`), a Nikto scan was run against the `/manager/html` directory on the Tomcat port:

<!-- SCREENSHOT: Nikto scan of /manager/html -->

- **Answer (number of documents):** 5
- **Answer (server version):** Apache/2.4.18
- **Answer (Apache-Coyote version):** 1.1 — the HTTP connector component inside Tomcat that lets the web server speak HTTP, receiving requests and translating them into a form Tomcat understands.

### How the Exploit Works: Tomcat Manager WAR Upload
Apache Tomcat ships with a Manager application at `/manager/html` that lets authenticated administrators deploy web applications by uploading a WAR (Web Application Archive) file. If an attacker has valid Manager credentials, as recovered here for `bob`, they can deploy a malicious WAR containing a JSP payload. Once deployed, requesting that application executes the payload on the server, yielding remote code execution. Because the Tomcat service on this machine runs as root, the resulting shell is a root shell. Metasploit's Tomcat Manager upload module automates building the WAR, authenticating, deploying it, and catching the shell.

### Getting a Shell
Using Metasploit with the recovered credentials to exploit the Tomcat Manager returned a shell:

<!-- SCREENSHOT: Metasploit module configuration -->

<!-- SCREENSHOT: Metasploit exploitation in progress -->

<!-- SCREENSHOT: shell obtained / verifying the user -->

- **Answer (user):** root

## Flag Capture
Exploiting the Tomcat Manager with `bob`'s credentials produced a shell running as root, from which the flag was read:
- `ff1fc4a81affcc7688cf89ae7dc6e0e1`

## Lessons Learned
- **Enumeration feeds exploitation.** Each tool handed the next step its input: directory discovery found `protected`, `hydra` cracked its login, and those same credentials unlocked the Tomcat Manager.
- **Reused credentials widen access.** The basic-auth password for `bob` also worked for the Tomcat Manager, turning a single cracked password into full compromise.
- **Exposed management interfaces are high-value targets.** A reachable Tomcat Manager with weak credentials is effectively remote code execution, since it is built to deploy applications.
- **Service accounts running as root are dangerous.** Because Tomcat ran as root, deploying one malicious WAR produced a root shell directly, with no separate privilege-escalation step needed.

## Tools Used
- `nmap`
- `gobuster` / `dirb`
- `hydra`
- `nikto`
- Metasploit (Tomcat Manager exploit module)
