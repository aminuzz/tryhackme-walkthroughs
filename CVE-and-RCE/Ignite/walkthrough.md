# Ignite -- TryHackMe Writeup

**Difficulty:** Easy  
**Category:** Web Exploitation / Linux Privilege Escalation

Note: The exploit used in this walkthrough was not written by me, so full credit goes to **p0dalirius**, the original author. The exploit is included in this repository, and the original can be found on [GitHub](https://github.com/p0dalirius/CVE-2018-16763-FuelCMS-1.4.1-RCE).

## Introduction
Ignite is an easy-rated TryHackMe room built around Fuel CMS 1.4, which is vulnerable to CVE-2018-16763, an unauthenticated remote code execution flaw. The chain is short but instructive: fingerprint the CMS during enumeration, use the known RCE to land a foothold and read the user flag, then escalate to root using a plaintext password left behind in one of the application's configuration files.

## Enumeration
### Nmap Scan
The first step was an `nmap` scan to identify the open ports and services:

<img width="803" height="191" alt="image" src="https://github.com/user-attachments/assets/32b77577-486f-40a1-8613-ea1efe3f67d7" />

### Web Enumeration
Visiting the site showed it was running Fuel CMS, specifically version 1.4:

<img width="1911" height="916" alt="image" src="https://github.com/user-attachments/assets/c07ac5fe-9e91-4b73-bb6f-5ce5750d572e" />

- Fuel CMS is an open-source, PHP-based content management system. Identifying the exact version is what makes matching it to a known CVE possible.

## Exploitation
### Finding the Exploit
There are several public exploits for this version, but the relevant one targets a remote code execution vulnerability. Before running anything, I looked into why the exploit works:

- The exploit covers **CVE-2018-16763**, a vulnerability that allows an **unauthenticated** attacker to execute PHP code on the machine, leading to full system compromise.
- The flaw lives in the `fuel/pages/select` endpoint, specifically in the `filter` parameter (it can also be reached through `preview/`). The root cause is improper input sanitization and validation, so the application ends up treating attacker input as PHP code, allowing command injection.

### Breaking Down the Exploit
In Burp Suite, the proof of concept sends a request to a URL like:

`http://<target>/fuel/pages/select/filter=%27%2Bpi%28print%28%24a%3D%27system%27%29%29%2B%24a%28%27ls%27%29%2B%27`

<img width="1918" height="916" alt="image" src="https://github.com/user-attachments/assets/c89f7b96-c20e-4108-afc4-b7cc3c357f52" />

Decoding the payload in CyberChef gives: `'+pi(print($a='system'))+$a('ls')+'`

Breaking that down:
- `$a='system'` stores the string `"system"` in the variable `$a`.
- `print($a='system')` assigns `"system"` to `$a` and prints it.
- `pi(...)` is mostly there to keep the whole expression valid in the vulnerable context; the important part is that the injected code gets evaluated.
- `$a('ls')` works because, in PHP, a variable holding a function name can be called like a function. So with `$a = 'system'`, `$a('ls')` is the same as `system('ls')`, running `ls` on the server's OS.
- The surrounding `'+ ... +'` breaks out of the string the vulnerable application is building internally, injects the attacker's PHP, then stitches the expression back together so the page does not immediately crash.

`ls` is used first simply to confirm code execution:
<img width="1914" height="872" alt="Gif29" src="https://github.com/user-attachments/assets/1dfc3d8c-0388-4d49-be39-34ae0866edb0" />


### Running the Exploit
I could not get the manual PoC to work reliably, so I used [p0dalirius's implementation](https://github.com/p0dalirius/CVE-2018-16763-FuelCMS-1.4.1-RCE) of the same CVE instead:
<img width="684" height="158" alt="image" src="https://github.com/user-attachments/assets/b93632fa-66c8-48be-b6ed-f17aad02d114" />

Running the script returned command execution on the target:
<img width="1646" height="611" alt="image" src="https://github.com/user-attachments/assets/8ae81735-932e-4ee3-a093-6ed3085c8355" />

- The shell does not support changing directories, so the flag had to be located manually rather than by navigating around.

## Gaining Access
Locating and reading the user flag:

<img width="456" height="582" alt="image" src="https://github.com/user-attachments/assets/564db736-0e18-477e-9a12-d07e94e65b72" />

## Privilege Escalation
### Background
To escalate, I first checked the sudoers configuration and looked through the file system, but neither offered an obvious path:

<img width="613" height="135" alt="image" src="https://github.com/user-attachments/assets/2cf4665c-a40b-49b4-9ab8-ad5361505ef9" />
<img width="748" height="556" alt="image" src="https://github.com/user-attachments/assets/d98db11d-56b4-4021-bc46-d0a3ba84b18f" />


To make enumeration easier, I set up a proper reverse shell back to my machine after some trial and error:

<img width="810" height="41" alt="image" src="https://github.com/user-attachments/assets/f5919236-b1a2-4a8f-b88e-0599990fb65f" />

### How the Exploit Works: Credentials in a Config File
Web applications need database credentials to run, and those are stored in configuration files on disk. Fuel CMS keeps its database settings in the application's config directory, and here the stored password had been reused for the root account and left in plaintext. Reading that file exposes a working root password, which is a classic privilege-escalation vector: hardcoded or reused credentials in a readable web-app config turn a low-privileged foothold into full root access.

Digging through the application, I found the root password:

<img width="986" height="212" alt="image" src="https://github.com/user-attachments/assets/2cc48ee1-2085-4dea-87e2-ea229cc2ecda" />

The credentials are stored in the Fuel CMS database configuration file:
<img width="428" height="355" alt="image" src="https://github.com/user-attachments/assets/d7e4907e-a555-46c0-99bf-bff2496cf864" />


### Escalating to root
Using the recovered password to switch to the root user granted a root shell and the root flag:
<img width="563" height="263" alt="image" src="https://github.com/user-attachments/assets/b7b5de8b-9174-4351-8f75-19d04910e39c" />

## Flag Capture
Both flags were recovered: the user flag from the CVE-2018-16763 RCE foothold, and the root flag after reusing the plaintext password found in Fuel CMS's configuration file to switch to root.

## Lessons Learned
- **Version fingerprinting leads straight to the exploit.** Identifying Fuel CMS 1.4 is what pointed directly at CVE-2018-16763. A visible version string is worth looking for.
- **Understand the exploit before running it.** Knowing the payload abuses PHP's variable-function behavior (`$a = 'system'; $a('ls')`) explains exactly how a filter parameter becomes command execution.
- **When one PoC fails, a maintained implementation often works.** The manual Burp PoC was unreliable, but p0dalirius's version of the same CVE landed cleanly.
- **Web-app config files hold secrets.** Database configuration files routinely contain plaintext credentials. Reading them should be a standard privilege-escalation check.
- **Password reuse is a high-impact mistake.** The database password doubling as the root password turned a limited web foothold into full root access.

## Tools Used
- `nmap`
- `searchsploit / Exploit-DB`
- **Burp Suite**
- **CyberChef**
- [**p0dalirius CVE-2018-16763 exploit**](https://github.com/p0dalirius/CVE-2018-16763-FuelCMS-1.4.1-RCE)
- `nc`
