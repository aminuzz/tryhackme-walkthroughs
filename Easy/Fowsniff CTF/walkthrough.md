# Fowsniff CTF -- TryHackMe Writeup

**Difficulty:** Easy  
**Category:** OSINT / Mail Services / Linux Privilege Escalation

## Introduction
Fowsniff CTF is an easy-rated TryHackMe room built around a fictional company that suffered a data breach. The attack chain leans on open-source intelligence and mail protocols: the website hints at a leak, a public GitHub repository holds the breached employee credentials, and cracking those hashes plus brute-forcing POP3 leads to a mailbox containing a temporary SSH password. Privilege escalation then abuses a login banner (MOTD) script that runs as root and is writable by a group the compromised user belongs to.

## Enumeration
### Nmap Scan
The first step was an `nmap` scan to identify open ports and services:
<img width="1213" height="410" alt="image" src="https://github.com/user-attachments/assets/e6034046-ea32-4e4c-a9f3-f223fdf6e363" />


- Port 22 is open for SSH.
- Port 80 is open for HTTP, hosting an Apache web server for "Fowsniff Corp - Delivering Solutions".
- Ports 110 and 143 are open for the POP3 and IMAP mail protocols. POP3 downloads email from a server (and typically deletes it), while IMAP syncs email across devices so it stays accessible everywhere.

### Web Enumeration
The website revealed that Fowsniff Corp had suffered a data breach that leaked employee usernames and passwords:
<img width="1913" height="921" alt="image" src="https://github.com/user-attachments/assets/5e01be76-6e12-495e-a992-25313b826f8f" />


A line of text on the site provided the clue needed for the next step:
<img width="1915" height="927" alt="image" src="https://github.com/user-attachments/assets/131000f1-4903-4734-81cf-7b504d003538" />

### OSINT: Finding the Leaked Credentials
A quick Google search surfaced a public GitHub repository containing the breached employee usernames and password hashes:

<img width="1915" height="922" alt="image" src="https://github.com/user-attachments/assets/67d0ad8e-abbf-401a-b52b-258214a4d43e" />


## Gaining Access
### Cracking the Hashes
The leaked passwords were stored as MD5 hashes, which were cracked using online lookup services such as [hashes.com](https://hashes.com/en/decrypt/hash) and [CrackStation](https://crackstation.net/):
<img width="1917" height="946" alt="image" src="https://github.com/user-attachments/assets/6c291fca-e1b5-4b88-8dbd-65bbbe8cdbeb" />
<img width="1915" height="940" alt="image" src="https://github.com/user-attachments/assets/73cca7f4-7516-4c65-98f9-f1111cd62267" />
<img width="1916" height="939" alt="image" src="https://github.com/user-attachments/assets/bb93f979-1edb-4f71-8b75-4fdbe5be4107" />
<img width="1918" height="945" alt="image" src="https://github.com/user-attachments/assets/6739ba23-4184-459b-bc1d-2316a0015dcf" />
<img width="1918" height="940" alt="image" src="https://github.com/user-attachments/assets/87040d9c-1d4b-4e2c-b2bc-16a17cdcc677" />
<img width="1917" height="948" alt="image (13)" src="https://github.com/user-attachments/assets/1602ddcf-1d9e-4974-afb2-1fd3c4c1b80a" />
<img width="1917" height="946" alt="image" src="https://github.com/user-attachments/assets/7d8557c0-d6eb-4476-a8b9-79bb7b7de7ea" />
<img width="1915" height="947" alt="image" src="https://github.com/user-attachments/assets/6704dd74-12d5-48e2-9d5e-786201312429" />

One hash (for `stone@fowsniff.com`) did not crack and was set aside.

### How the Exploit Works: POP3 Brute-Force
With a list of usernames and cracked passwords in hand, the next step was to find which pair actually worked against a live service. POP3 accepts username/password authentication, so the recovered credentials can be sprayed against it until one succeeds. Metasploit includes an auxiliary module that automates this POP3 credential check across the whole list:

<img width="1665" height="448" alt="image" src="https://github.com/user-attachments/assets/0f37774b-34c8-442a-a38d-3cd5d1623ae6" />


<img width="1915" height="922" alt="image" src="https://github.com/user-attachments/assets/ea68a594-828d-4e70-8ac4-b9b073f0fef8" />


- The working POP3 credentials were `seina:scoobydoo2`.

### Reading the Mailbox
Logging into the mail server with those credentials and reading the user's email revealed a temporary password that had been set for her:

<img width="1910" height="925" alt="image" src="https://github.com/user-attachments/assets/6e5ebaa9-8035-4376-9eb3-bcb23a9bbdde" />


This provided an SSH password. Logging in over SSH using the username of the person who sent the email:
<img width="1914" height="920" alt="image" src="https://github.com/user-attachments/assets/5c6f3a38-3af2-41b1-a7f3-7d238d81ab27" />


## Privilege Escalation
### How the Exploit Works: Writable MOTD Script
After logging in, checking the user's group memberships pointed to a file called `cube.sh` with interesting permissions:

<img width="638" height="32" alt="image" src="https://github.com/user-attachments/assets/1ba0ea3d-62d7-4faf-8e42-2b01d673441e" />


<img width="1915" height="923" alt="image" src="https://github.com/user-attachments/assets/cfb9ec04-efbf-4758-a680-70f679f2c463" />

<img width="848" height="516" alt="image" src="https://github.com/user-attachments/assets/21adbd4a-0d06-4ccd-9470-0b34d5ba31a7" />

- The key piece is in `/etc/update-motd.d/`. The MOTD (Message Of The Day) scripts in that directory run automatically as root every time a user logs in over SSH, to generate the login banner. The `00-header` script there calls `cube.sh`, and because the compromised user is in the `users` group, they can edit `cube.sh`. By inserting a reverse shell into `cube.sh` and reconnecting over SSH, the root-owned `00-header` executes the modified script as root, handing back a root shell.


### Exploiting It
A reverse shell was written into `cube.sh`, and on reconnecting over SSH the MOTD script executed it as root:
<img width="1905" height="920" alt="image" src="https://github.com/user-attachments/assets/e0af0a50-1e73-4f3a-9ab2-c520f35fad7c" />

- The first reverse shell attempt did not work, so a different one was used



## Flag Capture
With the root shell obtained on reconnection, the root flag was read:

<img width="1916" height="919" alt="image" src="https://github.com/user-attachments/assets/f59c3e15-14bc-40a8-83d0-d2c0047ad3a0" />

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
