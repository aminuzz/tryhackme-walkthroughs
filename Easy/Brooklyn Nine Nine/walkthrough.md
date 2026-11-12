# Brooklyn Nine Nine -- TryHackMe Writeup

**Difficulty:** Easy
**Category:** Web / Steganography / Linux Privilege Escalation

## Introduction
Brooklyn Nine Nine is an easy-rated TryHackMe room that walks through a full penetration testing workflow, from enumeration to root. It begins with an exposed FTP service and a bare-bones website, pivots into image steganography to recover a user's SSH password, and finishes with privilege escalation through a misconfigured sudo permission on the Nano editor. It is a solid end-to-end box for practicing how small findings across different services chain into full compromise.

## Enumeration
### Nmap Scan
The first step was an `nmap` scan to identify open ports and services:
<img width="1318" height="928" alt="image" src="https://github.com/user-attachments/assets/9bad6689-cd50-4d2c-88aa-81e4cf0973df" />

- Port 21 (FTP) is open and allows anonymous login, meaning a user can log in without a password. It contains a file called `note_to_jake.txt`.
- Port 22 (SSH) is open for remote login.
- Port 80 (HTTP) is hosting an Apache web server.

### Web Enumeration
Before digging into FTP, I checked the website:
<img width="1920" height="1044" alt="website_screenshot" src="https://github.com/user-attachments/assets/9bab2814-180f-4d20-a851-b94709628db3" />

The site is bare-bones, so the next move was to inspect the page source for clues:
<img width="1919" height="925" alt="source_code" src="https://github.com/user-attachments/assets/0cea67aa-2800-4062-8995-1606f56aa33c" />

- The source hints that the background image may be hiding data via steganography.
- It also gives away the image's file path: `brooklyn99.jpg`.

## Gaining Access
### How the Exploit Works: Steganography
Steganography is the practice of concealing data inside another file, commonly an image, so that nothing appears out of place to a casual viewer. Tools like `steghide` embed a payload behind a passphrase; the data is still present in the file, just hidden in a way that does not change the image's visible appearance. If the protecting passphrase is weak, it can be recovered with a dictionary attack, after which the hidden payload can be extracted. The comment in the page source pointing at the background image was the signal that this technique was in play.

### Extracting the Hidden Data
The next step was to download the image and attempt to extract its hidden contents with `stegseek`, which cracks the passphrase of a steghide-protected file using a wordlist (`rockyou.txt` by default) and extracts the payload in one pass:
<img width="1026" height="258" alt="image" src="https://github.com/user-attachments/assets/d624607b-d705-4929-80d0-2a680ed55411" />

<img width="934" height="251" alt="Screenshot_2026-05-21_09-59-19" src="https://github.com/user-attachments/assets/a5bfc3c7-15a6-4f52-9500-3830000af945" />

The output revealed:
- The passphrase protecting the data: `admin`
- The original hidden filename: `note.txt`
- The extracted output file: `brooklyn99.jpg.out`

Reading the extracted file gave up a user's credentials:
<img width="917" height="201" alt="holt_passwd" src="https://github.com/user-attachments/assets/9454de7c-2691-4960-8088-fd2f6613f98b" />

- This provided the password for the user `holt`, enough to access the machine over SSH.

### Checking the FTP Server
Before logging in, I circled back to the FTP server to read the file spotted earlier:
<img width="1920" height="659" alt="ftp" src="https://github.com/user-attachments/assets/b02eec4f-868f-4183-8b21-eecaf6a60640" />
<img width="1920" height="259" alt="Screenshot_2026-05-21_10-16-35" src="https://github.com/user-attachments/assets/951c63eb-cfd2-4de4-a7bb-5b9c9051542c" />

- This revealed another username, `jake`, worth keeping in mind.

### User Flag
Logging in as `holt` over SSH, the user flag was waiting in the home directory:
<img width="604" height="143" alt="user_flag" src="https://github.com/user-attachments/assets/08646d20-8063-481b-9757-8c5989069eea" />

## Privilege Escalation
### How the Exploit Works: Sudo on Nano
With a foothold as `holt`, the goal was to escalate to root. The first check in this situation is `sudo -l`, which lists the commands the current user may run with `sudo`. An overly permissive entry here is one of the most common and direct escalation paths: if a user can run an interactive program as root, and that program can spawn a shell or run arbitrary commands, then root is effectively one step away. The Nano editor is exactly such a program, it has a feature for executing external commands, so running it as root allows spawning a root shell.

### Exploiting Sudo on Nano
Checking sudo permissions showed that `holt` could run `/bin/nano` as any user, without a password:
<img width="1832" height="200" alt="sudoer_file" src="https://github.com/user-attachments/assets/3b0e84fc-f021-41dd-bc6b-8d09f429449a" />

That is the escalation vector. [GTFOBins](https://gtfobins.org/) documents how to abuse `nano` run under sudo to spawn a root shell:
<img width="1280" height="825" alt="nano" src="https://github.com/user-attachments/assets/26289ce9-b3c6-44a8-afa9-be3622fef1b5" />

Following that technique produced a root shell, and from there the root flag:
<img width="850" height="377" alt="root_flag" src="https://github.com/user-attachments/assets/a9c445e9-c3d5-457f-bb6e-cc858a46ad1b" />

## Flag Capture
Both flags were recovered: the user flag from `holt`'s home directory, and the root flag after escalating via the Nano sudo misconfiguration.

## Lessons Learned
- **Always start with proper enumeration.** Open ports and exposed services (FTP, HTTP, SSH) laid out the entire attack path before any exploitation began.
- **Anonymous FTP leaks context.** Even when a file is not the key itself, `note_to_jake.txt` surfaced another valid username, the kind of detail that matters on larger engagements.
- **Page source is enumeration.** A comment in the HTML pointed directly at the steganography angle and even named the image file.
- **Weak passphrases fall to wordlists.** The steghide passphrase (`admin`) was trivially cracked with `stegseek` against `rockyou.txt`.
- **Misconfigured sudo is a direct road to root.** Sudo access to an editor like `nano` is enough to spawn a root shell, and `sudo -l` plus GTFOBins is the fastest way to find and weaponize it.

## Tools Used
- `nmap` — Port scanning and service enumeration
- `ftp` — Accessing the anonymous FTP server
- `wget` — Downloading files from the target website
- `stegseek` — Cracking the steganography passphrase and extracting hidden files
- `ssh` — Remote login to the target machine
- `sudo` — Checking elevated command permissions
- `nano` — Abused for privilege escalation
- GTFOBins — Reference for Linux binary privilege escalation techniques
