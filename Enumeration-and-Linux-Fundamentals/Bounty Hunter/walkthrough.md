# Bounty Hacker -- TryHackMe Writeup

**Difficulty:** Easy  
**Category:** Enumeration / Linux Privilege Escalation

## Introduction
Bounty Hacker is a beginner-friendly, Cowboy Bebop themed TryHackMe room that walks through a classic enumeration-to-root attack chain. It begins with reconnaissance using `nmap` to map out the open ports and services, revealing FTP, SSH, and a web server. Anonymous FTP access then exposes a pair of text files, a note signed by a user and a wordlist of potential passwords, which together set up an SSH brute-force with `hydra` to recover valid credentials and gain an initial foothold. After capturing the user flag, the room finishes with privilege escalation by running `sudo -l` to spot a misconfigured `tar` binary that can be executed as root, then abusing it to spawn a root shell.

## Enumeration
### Nmap Scan
The first step was an `nmap` scan to find the open ports and services running on the machine:
```
nmap -sV -sC -T4 10.64.173.227
```
<img width="1920" height="965" alt="nmap_scan" src="https://github.com/user-attachments/assets/b091e122-7db8-4960-afd2-cf62cc2ac0e9" />

- Port 21 is running an FTP server, version vsftpd 3.0.5.
- Port 22 is running SSH, specifically OpenSSH 8.2p1.
- Port 80 is running an Apache web server, version 2.4.41.

### FTP Enumeration
Before investigating the web server, I checked the FTP service, since it allows anonymous login, meaning a user can access the files on the server without valid credentials:
```
ftp anonymous@10.64.173.227
```

Listing the contents revealed two files:
<img width="956" height="432" alt="ftp" src="https://github.com/user-attachments/assets/d7ae2853-8939-46b2-b2ec-a32548559551" />

I downloaded both to my working directory for investigation:
<img width="962" height="389" alt="downloading_files_to_directory" src="https://github.com/user-attachments/assets/f35b9119-6c11-4a71-b504-113d1deda306" />

- `get` downloads a file from the FTP server to your machine, saving it to whichever local directory you launched the FTP session from.

The first file, `locks.txt`, had the shape of a password list:
<img width="959" height="764" alt="locks" src="https://github.com/user-attachments/assets/d71e6351-0c5b-48ec-a80f-a87ff50c62cc" />

- Based on the patterns in each line, this looks like a list of candidate passwords for logging into the machine, the website, or another service.

The second file, `task.txt`, handed over a likely username:
<img width="960" height="167" alt="task" src="https://github.com/user-attachments/assets/cf8c406c-73b9-42eb-bf79-a88d65911eeb" />

- The content is mostly background story, but it is signed by `lin`, a strong candidate for a username on the target machine.

### Web Enumeration
Investigating the website produced nothing useful, even after a directory enumeration scan:
<img width="1920" height="1002" alt="website_ss" src="https://github.com/user-attachments/assets/0bf28bb9-3512-4a75-8863-527486ce0c44" />

With the web path a dead end, the right move was to pivot back to the data already in hand.

## Gaining Access
### Background
Two halves of a credential attack were now available: a wordlist (`locks.txt`) and a likely username (`lin`). `hydra` automates a brute-force by testing each password in a wordlist against a login service until one succeeds. Recognizing when you have been handed both a username and a wordlist is the cue to reach for a brute-force tool rather than keep enumerating.

### Brute-Forcing SSH
`hydra` was used to brute-force `lin`'s SSH password against the wordlist:
```
hydra -l lin -P locks.txt ssh://10.64.173.227
```
<img width="958" height="522" alt="hydra" src="https://github.com/user-attachments/assets/fe413bf3-d7be-4ad0-8ed9-14a04318d3c8" />

With the recovered credentials I logged in over SSH and grabbed the first flag:
<img width="957" height="123" alt="flag1" src="https://github.com/user-attachments/assets/c8dc1c4c-4f55-417b-a53b-bb59426d9fc7" />

## Privilege Escalation
### How the Exploit Works: Sudo on tar
With a valid password in hand, the fastest privilege-escalation check is `sudo -l`, which lists what the current user may run as root. `tar` turning up here is significant because of two of its options: `--checkpoint` triggers an action at set intervals during archive processing, and `--checkpoint-action=exec=` specifies a command to execute when that checkpoint is hit. Together, these let `tar` run an arbitrary command as part of a normal archive operation. When `tar` is run as root via sudo, any command it executes through a checkpoint action inherits root's privileges, which turns a legitimate archiving tool into a root shell.

### Exploiting Sudo on tar
Checking sudo permissions confirmed the user could run `/bin/tar` as root:
```
sudo -l
```
<img width="953" height="261" alt="f" src="https://github.com/user-attachments/assets/18d2a601-83c8-4f8e-8323-276c7636d747" />

Referencing [GTFOBins](https://gtfobins.org/gtfobins/tar/#shell), I used the checkpoint-action technique to spawn a root shell:
```
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```
<img width="960" height="259" alt="root_flag" src="https://github.com/user-attachments/assets/254ec358-7677-48b9-a315-76c858823cac" />

## Flag Capture
Both flags were recovered: the user flag after the `hydra`/SSH login as `lin`, and the root flag once the `tar` sudo misconfiguration was abused to spawn a root shell.

## Lessons Learned
- **Enumerate every service, not just the web server.** The website was a complete dead end; the real foothold came from the FTP server. Checking each open port the `nmap` scan reported is what surfaced the anonymous FTP access.
- **Anonymous FTP is a feature; exposed files are the flaw.** Anonymous login is not a vulnerability by itself. The actual weakness was a security misconfiguration: sensitive files left readable in the anonymous-accessible directory.
- **Read the "boring" files anyway.** `task.txt` looked like throwaway story flavor, but its signature handed over a likely username. Names, emails, and signatures are often deliberate breadcrumbs toward credentials.
- **Dead ends are part of the process.** Web enumeration returned nothing, but rather than forcing that path, the right move was to pivot back to the wordlist already in hand and attack a different service.
- **Run sudo -l first for privilege escalation.** Once you have a valid password, checking what the user can run as root is the fastest, lowest-effort privesc check, and here it immediately revealed the `tar` misconfiguration.
- **GTFOBins is the first stop for binary abuse.** A binary runnable as root via sudo (like `tar`) is a classic, high-impact misconfiguration with a documented escalation path.

## Tools Used
- `nmap`
- `hydra`
- `ffuf`
- [GTFOBins](https://gtfobins.org/)
