# Fowsniff CTF -- TryHackMe Writeup

**Difficulty:** Easy  
**Category:** OSINT / Mail Services / Linux Privilege Escalation

## Introduction
Fowsniff CTF is an easy-rated TryHackMe room built around a fictional company that suffered a data breach. The attack chain leans on open-source intelligence and mail protocols: the website hints at a leak, a public GitHub repository holds the breached employee credentials, and cracking those hashes plus brute-forcing POP3 leads to a mailbox containing a temporary SSH password. Privilege escalation then abuses a login banner (MOTD) script that runs as root and is writable by a group the compromised user belongs to.

## Enumeration
### Nmap Scan
The first step was an `nmap` scan to identify open ports and services:

<!-- SCREENSHOT: nmap scan output -->

- Port 22 is open for SSH.
- Port 80 is open for HTTP, hosting an Apache web server for "Fowsniff Corp - Delivering Solutions".
- Ports 110 and 143 are open for the POP3 and IMAP mail protocols. POP3 downloads email from a server (and typically deletes it), while IMAP syncs email across devices so it stays accessible everywhere.

### Web Enumeration
The website revealed that Fowsniff Corp had suffered a data breach that leaked employee usernames and passwords:

<!-- SCREENSHOT: website announcing the data breach -->

A line of text on the site provided the clue needed for the next step:

<!-- SCREENSHOT: clue text on the website -->

### OSINT: Finding the Leaked Credentials
A quick Google search surfaced a public GitHub repository containing the breached employee usernames and password hashes:

<!-- SCREENSHOT: GitHub repository with leaked employee credentials -->

## Gaining Access
### Cracking the Hashes
The leaked passwords were stored as MD5 hashes, which were cracked using online lookup services such as [hashes.com](https://hashes.com/en/decrypt/hash) and [CrackStation](https://crackstation.net/):

<!-- SCREENSHOT: cracking the MD5 hashes (1) -->

<!-- SCREENSHOT: cracking the MD5 hashes (2) -->

<!-- SCREENSHOT: cracking the MD5 hashes (3) -->

<!-- SCREENSHOT: cracked credentials list -->

One hash (for `stone@fowsniff.com`) did not crack and was set aside.

### How the Exploit Works: POP3 Brute-Force
With a list of usernames and cracked passwords in hand, the next step was to find which pair actually worked against a live service. POP3 accepts username/password authentication, so the recovered credentials can be sprayed against it until one succeeds. Metasploit includes an auxiliary module that automates this POP3 credential check across the whole list:

<!-- SCREENSHOT: Metasploit POP3 auxiliary module configuration -->

<!-- SCREENSHOT: Metasploit POP3 brute-force result -->

- The working POP3 credentials were `seina:scoobydoo2`.

### Reading the Mailbox
Logging into the mail server with those credentials and reading the user's email revealed a temporary password that had been set for her:

<!-- SCREENSHOT: logging into the mailbox and reading the temporary password -->

This provided an SSH password. Logging in over SSH using the username of the person who sent the email:

<!-- SCREENSHOT: SSH login using the email sender's username -->

## Privilege Escalation
### How the Exploit Works: Writable MOTD Script
After logging in, checking the user's group memberships pointed to a file called `cube.sh` with interesting permissions:

<!-- SCREENSHOT: group membership and cube.sh permissions (1) -->

<!-- SCREENSHOT: cube.sh permissions (2) -->

The key piece is in `/etc/update-motd.d/`. The MOTD (Message Of The Day) scripts in that directory run automatically as root every time a user logs in over SSH, to generate the login banner. The `00-header` script there calls `cube.sh`, and because the compromised user is in the `users` group, they can edit `cube.sh`. By inserting a reverse shell into `cube.sh` and reconnecting over SSH, the root-owned `00-header` executes the modified script as root, handing back a root shell.

### Exploiting It
A reverse shell was written into `cube.sh`, and on reconnecting over SSH the MOTD script executed it as root:

<!-- SCREENSHOT: editing cube.sh with a reverse shell -->

The first reverse shell attempt did not work, so a different one was used:

<!-- SCREENSHOT: working reverse shell / root connection -->

## Flag Capture
With the root shell obtained on reconnection, the root flag was read:

<!-- SCREENSHOT: reading the root flag -->

## Lessons Learned
- **Breach data is OSINT.** The entire foothold came from a public GitHub repository of leaked credentials that the website hinted at. Following the clue to its source is the core skill this room teaches.
- **Spray recovered credentials against live services.** Cracked hashes are only useful once validated. A POP3 brute-force identified which single credential pair actually worked.
- **Mailboxes hold secrets.** The temporary SSH password was sitting in an email, a reminder to read every accessible mailbox during enumeration.
- **MOTD scripts run as root on login.** Anything in `/etc/update-motd.d/` executes with root privileges at SSH login. A writable script there (or one it calls) is a direct, reliable privilege-escalation path.

## Tools Used
- `nmap`
- hashes.com / CrackStation
- Metasploit (POP3 auxiliary module)
- `ssh`
