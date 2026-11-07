# Lazy Admin -- TryHackMe Writeup

**Difficulty:** Easy  
**Category:** Web / Linux Privilege Escalation

## Introduction
Lazy Admin is an easy rated TryHackMe room built around a classic chain of small misconfigurations rather than a single flashy exploit. The vulnerable application is **SweetRice CMS**, an older PHP content management system, and the room tests whether you can enumerate past the default Apache landing page to find it, locate an exposed backup file that leaks admin credentials, crack the resulting password hash, and use those credentials to abuse a file upload vulnerability for remote code execution. Privilege escalation then hinges on a misconfigured `sudo` permission on a Perl script, which chains into a writable shell script that can be hijacked to deliver a root shell.


### Quick Disclaimer
You'll notice that the IP addresses are different in different commands, and that's because I worked on the challenge across multiple sessions, with the IP address changing each time.

## Enumeration
### Nmap Scan
The first step was to run an `nmap` scan to identify open ports and services running on the machine:
```bash
nmap -sV -sC -T4 10.65.147.221
```
- `-sV` obtains the service version of each open port.
- `-sC` runs `nmap`'s default scripts to gather additional information about the services running.
- `-T4` uses aggressive timing to speed up the scan (in a real-world scenario, this would be noisy and easily flagged).

<img width="1913" height="513" alt="nmap_scan" src="https://github.com/user-attachments/assets/96e402ec-5cd4-45c9-bee8-e36c6890ed25" />

- Port 22 is running `ssh`, specifically **OpenSSH 7.2p2**.
- Port 80 is running `http`, specifically **Apache 2.4.18**.

### Web Enumeration
Based on the `nmap` scan, the landing page was the Apache2 default web page:
<img width="1920" height="1003" alt="default_webpage" src="https://github.com/user-attachments/assets/f39b1b55-bfc1-47cd-90a5-51e7f94e941b" />

Since the default page gave nothing away, the next step was to assume hidden directories existed and fuzz for them using `ffuf`:
```bash
ffuf -u http://10.64.186.158/FUZZ -w /usr/share/wordlists/dirb/big.txt -e .js,.py,.php,.txt,.html,.css -mc 200,301,302
```
<img width="1920" height="775" alt="web_enumeration" src="https://github.com/user-attachments/assets/2af6b93b-76e5-4f38-8c70-d3e7733dc973" />

## Gaining Access
### Background
The initial `ffuf` scan surfaced a hidden directory, `content`. The page itself held nothing useful at first glance, and checking the page source for hardcoded secrets came up empty, so the next step was to enumerate further within that directory using `gobuster`.
<img width="1484" height="919" alt="image(4)" src="https://github.com/user-attachments/assets/0a57bf81-c364-453a-9342-d195583ff9ea" />
<img width="966" height="542" alt="image(6)" src="https://github.com/user-attachments/assets/39c40be5-2afb-4377-88c1-fb40149f9430" />



That scan returned a set of additional pages, most of which pointed into the CMS's own file structure and weren't directly accessible. One file stood out, however a version number that identified the CMS as **SweetRice**.

<img width="1473" height="924" alt="image(5)" src="https://github.com/user-attachments/assets/387231ea-3831-4add-b70f-4e30a9f5d6ea" />


With a specific CMS and version in hand, a quick search for known exploits turned up several relevant entries:

<img width="1454" height="245" alt="image(7)" src="https://github.com/user-attachments/assets/b3926564-3ee2-47c1-94d4-521f279bf069" />


- **Arbitrary File Download** — occurs when a web application fails to sanitize user input used in file download or retrieval functions, allowing an attacker to pull sensitive files directly off the server.
- **Arbitrary File Upload** — allows a user to upload any file to the server without restriction, which can lead to full compromise if that file is something like a reverse shell.
- **Backup Disclosure** — a vulnerability where the web server unintentionally exposes backups of sensitive files publicly, giving an attacker direct access to data that should never have been reachable.

Separately, a login portal was found at `http://10.65.139.73/content/as/`. An attempt to brute-force it with `hydra` returned multiple "valid" passwords for `admin`, a clear sign of a misconfigured login form producing false positives rather than a real result, so this path was abandoned in favor of the exploits already identified.
<img width="1474" height="687" alt="image(8)" src="https://github.com/user-attachments/assets/f3a1a74e-516a-4250-9c22-bb70423492ad" />


### Discovering the Backup File
The **Backup Disclosure** vulnerability was the most promising lead. Visiting the disclosed path allowed direct access to the CMS's database backup and other website backup files:

<img width="631" height="333" alt="image(9)" src="https://github.com/user-attachments/assets/9257cf5d-26bc-4f5e-8189-4850f8775f57" />


Downloading and inspecting the SQL dump revealed a hashed password belonging to the site's admin/manager account:

<img width="1912" height="870" alt="image(10)" src="https://github.com/user-attachments/assets/85b65bc2-02d4-4e79-86bb-dd231d3f3eb3" />

### Cracking the Hash
The hash was cracked, recovering a plaintext password:

<img width="758" height="173" alt="image(11)" src="https://github.com/user-attachments/assets/b8ef530c-baaa-4d3f-9f19-254c9ac27a77" />

### Logging In and Exploiting the File Upload Vulnerability
With the cracked password in hand, a login attempt using the username `admin` failed, but `manager` succeeded, since the SweetRice CMS evidently stores its administrative account under that username rather than `admin`.
<img width="1915" height="922" alt="image(12)" src="https://github.com/user-attachments/assets/8d55ca4e-71db-4f6f-a100-1755196169fd" />


With access to the admin panel, the next step was to use the **Arbitrary File Upload** vulnerability to get a shell on the underlying server. An initial attempt to upload a PHP reverse shell directly was blocked, the CMS was filtering `.php` uploads specifically.

<img width="1907" height="914" alt="image(13)" src="https://github.com/user-attachments/assets/43aa1481-373e-4e13-8cac-4f6153c3e64a" />


Testing which file types were accepted showed that the filter was extension based rather than content based, so the reverse shell was simply renamed to use a `.php5` extension instead, an alternate PHP file extension that many servers still execute but that the upload filter didn't account for.

<img width="1912" height="834" alt="image(15)" src="https://github.com/user-attachments/assets/2d551be5-6ce5-474f-abd6-56e671d26733" />


This bypassed the restriction and triggered the shell, granting a foothold on the machine:

<img width="940" height="238" alt="image(16)" src="https://github.com/user-attachments/assets/2b3dbd68-4045-4e59-9070-4b1fc40820b0" />



The shell was then upgraded to a fully interactive TTY, and the user flag was located shortly after:

<img width="452" height="17" alt="image(17)" src="https://github.com/user-attachments/assets/f2456506-9692-45f4-8f5b-c1f69858fa9a" />
<img width="369" height="69" alt="image(18)" src="https://github.com/user-attachments/assets/26b6600e-6c1b-4fca-9517-a2c178984c4f" />


## Privilege Escalation
### Background
The SQL credentials found earlier during enumeration were also present in the same directory as the user flag, but attempting to use them to access the database directly led nowhere. With that path closed off, the next step was to check `sudo -l` to see what the current user could run as a higher privilege.

<img width="770" height="102" alt="image(20)" src="https://github.com/user-attachments/assets/d34d4cd4-d218-4631-8989-dd3880956df5" />


The output showed that `perl` could be run with `sudo`. A `perl` script was also present in the current directory, so it was worth examining:

<img width="429" height="69" alt="image(21)" src="https://github.com/user-attachments/assets/8f85ca47-6a8e-40a3-8c8a-870551376c65" />

The script ran two system-level commands, one of which called `sh` to execute `/etc/copy.sh`. Critically, `/etc/copy.sh` was owned by `root`, but the script itself was writable by the current, lower-privileged user, meaning its contents could be swapped out for something else entirely, which would still execute with root's permissions once triggered through the sudo-enabled `perl` script.

<img width="700" height="61" alt="image(22)" src="https://github.com/user-attachments/assets/040e91ad-c06b-443d-ae5d-fd3067a041e1" />

### Exploiting the Sudo Misconfiguration
Since `/etc/copy.sh` was already structured as a reverse shell, it only needed to be edited to point at the attacking machine's IP and listening port:
<img width="859" height="30" alt="image(23)" src="https://github.com/user-attachments/assets/2681b635-cd87-4318-a596-648685adb30c" />


Running the `perl` script through `sudo` (matching the exact syntax listed in the sudoers file, since any deviation causes it to fail) then triggered `/etc/copy.sh` as `root`, delivering a root-level reverse shell:
<img width="585" height="28" alt="image(24)" src="https://github.com/user-attachments/assets/c1a07b57-b4e3-48fa-a69e-093cf0ccb781" />


## Flag Capture
With a root shell established, the final flag was retrieved, completing the room.
<img width="340" height="58" alt="image(25)" src="https://github.com/user-attachments/assets/f8387fb3-e2f7-480f-8ffd-60130393852f" />


## Lessons Learned
- **A version number is often all it takes.** A single file exposing the CMS's version string was the thread that unraveled the entire chain, from identifying known exploits to eventually finding a working foothold.
- **Don't trust a brute-force "success."** Hydra returning multiple valid passwords for the same account was a red flag for a misconfigured login form, not a real result. Recognizing false positives early saved time that would have been wasted chasing a dead end.
- **File upload filters are often shallower than they look.** The CMS blocked `.php` specifically but not alternate PHP-executable extensions like `.php5`, a reminder that extension based filtering is rarely a complete defense.
- **Writable files owned by root, triggered by sudo, are a direct path to root.** The real vulnerability wasn't `perl` itself, it was that a root-owned script called a second script that an unprivileged user could edit. Any time a privileged process executes something attacker-controlled, that's the actual escalation point.
- **Exact syntax matters with sudo.** Deviating from the precise command listed in the sudoers file broke the exploit, a good reminder that `sudo` permissions are often scoped down to an exact invocation, not just a binary name.

## Tools Used
- `nmap`
- `ffuf`
- `gobuster`
- `hydra`
- `john`
