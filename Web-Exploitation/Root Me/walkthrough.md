# Root Me -- TryHackMe Writeup

**Difficulty:** Easy  
**Category:** Web Exploitation / Linux Privilege Escalation

## Introduction
Root Me is an easy-rated TryHackMe room centered on web exploitation and privilege escalation. It begins with reconnaissance using `nmap` to map open ports and services, followed by directory enumeration to uncover a hidden file-upload page. From there, an upload filter is bypassed to deliver a PHP reverse shell and gain an initial foothold. After capturing the user flag, the room walks through privilege escalation by identifying a misconfigured SUID binary and abusing it to spawn a root shell.

## Enumeration
### Nmap Scan
Once connected to the TryHackMe network, the first step was an `nmap` scan to see what the machine was running:
```
nmap -sV -sC -p- -T4 10.64.143.151
```
<img width="935" height="627" alt="nmap_scan" src="https://github.com/user-attachments/assets/2702547d-36e3-46eb-a50e-044c2f209fe6" />

- Port 22 is open running SSH, version 8.2p1.
- Port 80 is open running HTTP, Apache 2.4.41.
- The website title is HackIT - Home.

Visiting the website:
<img width="960" height="1002" alt="image" src="https://github.com/user-attachments/assets/fa3ccf9a-be18-4e49-a7c6-726417f024f5" />

### Directory Enumeration
The landing page was bland and gave no clues, so I enumerated for hidden pages with `ffuf`:
```
ffuf -u http://10.67.162.239/FUZZ -w /usr/share/wordlists/dirb/big.txt -e .js,.css,.txt,.php,.py,.html -mc 200,301,302
```
- The IP address differs from the previous command because the earlier session timed out, but the approach is identical: finding hidden directories and pages to gain access to the machine.

The scan found the following pages:
<img width="1916" height="153" alt="webpages_ffuf" src="https://github.com/user-attachments/assets/bc4bcb74-f545-44f1-9e5d-4eed49098e5a" />

### Reviewing the Pages
I examined each directory for an attack vector. First, the `css` directory:
<img width="954" height="1002" alt="css_directories" src="https://github.com/user-attachments/assets/72e39d24-4b09-4ffc-8923-bb10ae3973c6" />

- This contains `home.css` and `panel.css`, the files responsible for the site's layout, not useful as an attack vector.

Next, `index.php`:
<img width="962" height="998" alt="index_php_ss" src="https://github.com/user-attachments/assets/f4fb2183-5d8b-4a65-9ea7-0070cd7df4a3" />

- Nothing of interest beyond confirming the site uses PHP on the backend.

The `js` directory:
<img width="961" height="997" alt="js_ss" src="https://github.com/user-attachments/assets/0c1ef362-cddc-4d1b-b981-5c0356fe9acf" />

- Only `maquina_de_escrever.js` (Portuguese for "typewriter.js"), a plugin that produces a typewriter text effect on the page.

The `panel` page:
<img width="958" height="999" alt="panel_ss" src="https://github.com/user-attachments/assets/909bf36f-9097-40c2-9c33-2faa215ccdb3" />

- This page allows file uploads, which can be abused if the upload mechanism does not sanitize what is uploaded.
- Notably, `panel` was reachable without any authentication, an example of broken access control, since an anonymous user can reach an admin-style upload page.

Finally, `uploads`:
<img width="959" height="1005" alt="upload_ss" src="https://github.com/user-attachments/assets/90c4055e-830f-4f62-a9e6-88696fc524d0" />

- This is where uploaded files are stored on the server.

## Gaining Access
### How the Exploit Works: Unrestricted File Upload to Reverse Shell
With an upload page in hand, the goal was an unrestricted file upload attack. If an upload form fails to validate a file's content, type, or extension, an attacker can upload a reverse shell, a script that, when executed by the server, connects back to the attacker and hands over a command shell. Because the uploaded file lands in a web-accessible directory and the server executes PHP, simply requesting the file triggers remote code execution.

A reverse shell works like an SSH connection with the roles reversed: instead of the attacker connecting in, the target initiates an outbound connection back to the attacker over TCP. A listener such as `netcat` must be running first to catch that connection, after which the attacker is dropped into a basic `sh` session on the target.

### Uploading the Shell
Since the backend is PHP, I used [pentestmonkey's PHP reverse shell](https://pentestmonkey.net/tools/web-shells/php-reverse-shell) (also available on Kali under `/usr/share/webshells/php`), editing it to point at my IP, and set up a listener first:
```
nc -lnvp 1234
```
- `-l` listen, `-v` verbose, `-n` skip DNS resolution, `-p` port. The port must match the one set in the script.
<img width="962" height="1000" alt="shell_upload" src="https://github.com/user-attachments/assets/e596e33c-484f-4cb1-8636-134a4edb1351" />

Uploading the file returned an error:
<img width="961" height="1002" alt="error" src="https://github.com/user-attachments/assets/bdc718a2-6625-4fde-8b74-4d2685def8d2" />

- The text translates to "PHP is not allowed!"

This meant one of two things: the filter blocks all files containing `php`, or it blocks only the `.php` extension specifically. A quick search surfaced a [list of alternate PHP extensions](https://vulp3cula.gitbook.io/hackers-grimoire/exploitation/web-application/file-upload-bypass):
<img width="783" height="123" alt="alternate_extensions" src="https://github.com/user-attachments/assets/745570ba-9b85-40cf-a7b3-bc59a114cb67" />

I renamed the shell to use `.php5` and the upload succeeded:
<img width="957" height="1002" alt="success" src="https://github.com/user-attachments/assets/3a0c70ae-dc82-428d-8d3c-d6251bc07d14" />
<img width="960" height="1001" alt="upload_success" src="https://github.com/user-attachments/assets/3012aef4-4178-4d49-8aec-c3698d853e0e" />

- Apache is commonly configured to execute several PHP-related extensions as PHP code, but the upload filter only checked for `.php` specifically. `.php5` bypasses the filter entirely while Apache still hands the file to the PHP interpreter for execution. Whenever you hit an extension blocklist, look up alternate extensions the server's runtime may execute.

Navigating to the `uploads` directory and clicking the uploaded file makes the page hang, which is normal: the server has opened an outbound connection back to the listener, dropping into an `sh` session:
<img width="1902" height="331" alt="sh_session" src="https://github.com/user-attachments/assets/e7e220e5-c54b-4e12-8fb7-e9472eca9ee4" />

I then upgraded the reverse shell for better usability (this requires Python on the target):
<img width="801" height="345" alt="shell_upgrade" src="https://github.com/user-attachments/assets/61459a7d-e33a-49e4-871f-614e8efe6983" />

With the shell cleaned up, the first flag was in `/var/www/user.txt`:
<img width="1283" height="190" alt="flag_1" src="https://github.com/user-attachments/assets/103f2b5a-8786-4b74-ae15-1aa99b4c5ec9" />

## Privilege Escalation
### How the Exploit Works: SUID Binaries
With a low-privileged reverse shell and no knowledge of the user's sudo password, `sudo -l` was not an option. The next step was to check for binaries with the SUID bit set. The SUID bit causes a file to execute with the permissions of its owner rather than the user running it. If a binary owned by root has the SUID bit set and can run arbitrary code (for example, an interpreter like Python), an unprivileged user can use it to execute commands as root, escalating to full system control.

To find SUID binaries:
```
find / -type f -perm -4000 2>/dev/null
```
<img width="783" height="839" alt="SUID" src="https://github.com/user-attachments/assets/2c6ec4e6-ad73-4f54-829a-23dd8e5e00c5" />
<img width="849" height="68" alt="python_ownedbyroot" src="https://github.com/user-attachments/assets/5bb8466c-7618-4a81-a4ed-205312ccd87b" />

### Exploiting the SUID Python Binary
The binary that stood out was `/usr/bin/python2.7`, owned by root. [GTFOBins](https://gtfobins.org/) documents how a SUID Python binary can spawn a subshell that inherits root privileges:
<img width="958" height="381" alt="exploit_for_python" src="https://github.com/user-attachments/assets/52b6893c-329f-449e-859c-84d8d57bca1c" />

Running that command escalated to root and gave up the root flag:
<img width="1024" height="745" alt="root_flag" src="https://github.com/user-attachments/assets/a0e1a488-0c48-4999-8484-4284ea21b5b5" />

## Flag Capture
Both flags were recovered: the user flag from `/var/www/user.txt` after catching the PHP reverse shell, and the root flag after abusing the SUID-set `/usr/bin/python2.7` binary to spawn a root shell.

## Lessons Learned
- **Enumerate broadly, not just for .php.** Fuzzing with several extensions is what surfaced `panel` and `uploads`. The bland landing page gave nothing away, and a narrow wordlist would have missed the whole attack path.
- **Fingerprint the backend early.** Confirming the site ran on PHP directly informed the payload choice, saving time guessing at what would execute.
- **Upload filters are often shallow.** A "PHP is not allowed!" message rarely means all PHP is blocked; it usually blocks the `.php` extension specifically. Trying alternate extensions (`.php5`, `.phtml`) is a quick, high-value bypass.
- **Understand how a reverse shell works.** The roles are reversed: the target connects back to you, so the listener must be running first, and the script's IP must point at your machine. The page hanging after triggering the shell is expected.
- **Stabilize your shell before real work.** Upgrading the raw `sh` session to a full TTY made enumeration and privesc far smoother, though it depends on the target having Python.
- **When sudo -l is off the table, pivot.** A reverse shell gives a session without the user's password, so SUID enumeration with `find ... -perm -4000` is the natural next move. On a harder target, automated tools like `linpeas` or `LinEnum` would be worth running to catch any easier paths.
- **SUID misconfigurations are a fast track to root.** A single owner-root binary with the SUID bit set (here, Python) was enough to spawn a root shell, and GTFOBins documents exactly how to abuse common binaries this way.

## Tools Used
- `nmap`
- `ffuf`
- `nc`
- [pentestmonkey PHP reverse shell](https://pentestmonkey.net/tools/web-shells/php-reverse-shell)
- [GTFOBins](https://gtfobins.org/)
- [Hacker's Grimoire](https://vulp3cula.gitbook.io/hackers-grimoire)
