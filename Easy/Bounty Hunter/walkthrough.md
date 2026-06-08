# Bounty Hacker 💀
This TryHackMe room is a beginner-friendly, Cowboy Bebop themed challenge that walks through a classic enumeration-to-root attack chain. It begins with reconnaissance using `nmap` to map out the open ports and services, revealing ftp, ssh, and a web server. From there, anonymous FTP access exposes a pair of text files — a note signed by a user and a wordlist of potential passwords — which together set up an `ssh` brute-force attack using `hydra` to recover valid credentials and gain an initial foothold. After capturing the user flag, the room finishes with privilege escalation by running `sudo -l` to spot a misconfigured `tar` binary that can be executed as root, then abusing it via GTFOBins to spawn a root shell, reinforcing core concepts like service enumeration, anonymous FTP, credential brute-forcing, and sudo-based privilege escalation along the way.

## Reconnaissance 🔎
The first step was to perform a `nmap` scan on the machine to find open ports and services that were running on the machine:
```
nmap -sV -sC -T4 10.64.173.227
```
<img width="1920" height="965" alt="nmap_scan" src="https://github.com/user-attachments/assets/b091e122-7db8-4960-afd2-cf62cc2ac0e9" />
- port 21 has an `ftp` server running version **vsftpd 3.0.5**
- port 22 in typical fashion running `ssh` specifically `OpenSSH 8.2p1`
- port 80 is running an Apache web server (`http`) specifically version **2.4.41**

Before diving into the web server, I checked out the `ftp` server since it allows for an **Anonymous FTP login** which means a user can access the files located on the server without needing valid credentials:
```
ftp anonymous@10.64.173.227
```

After logging in, I listed the contents of the server and found two files:
<img width="956" height="432" alt="ftp" src="https://github.com/user-attachments/assets/d7ae2853-8939-46b2-b2ec-a32548559551" />

I then downloaded both files to my working directory and investigated further:
<img width="962" height="389" alt="downloading_files_to_directory" src="https://github.com/user-attachments/assets/f35b9119-6c11-4a71-b504-113d1deda306" />
- `get` is the command to use when you want to download a file from an `ftp` server to your machine but keep in mind it will download the file in the last working directory you were in

### FTP Files 📁
I first took a look at `locks.txt`:
<img width="959" height="764" alt="locks" src="https://github.com/user-attachments/assets/d71e6351-0c5b-48ec-a80f-a87ff50c62cc" />
- Based on the patterns of each line of text, this looks to be a list of passwords that could be used to login on the machine, the website, etc


Next was `task.txt`:
<img width="960" height="167" alt="task" src="https://github.com/user-attachments/assets/cf8c406c-73b9-42eb-bf79-a88d65911eeb" />
- Nothing of interest, probably something to do with the background story of the challenge however we are given a name `lin` which could be a username of the user account that we'll take advantage of

The next step was to investigate the website:
<img width="1920" height="1002" alt="website_ss" src="https://github.com/user-attachments/assets/0bf28bb9-3512-4a75-8863-527486ce0c44" />
- The website didn't have anything worthwhile to look at even after conducting a web enumeration scan so it was back to the drawing board




## Gaining Access 🔓
My next thought was to check if the passwords can be used to brute force something. With the web enumeration scan returning nothing, I pivoted back to the FTP files. `locks.txt` had the shape of a password list, and `task.txt` had handed me a likely username, `lin`, using `hydra` I was able to confirm the login for the machine:
```
hydra -l lin -P locks.txt ssh://10.64.173.227
```

<img width="958" height="522" alt="hydra" src="https://github.com/user-attachments/assets/fe413bf3-d7be-4ad0-8ed9-14a04318d3c8" />

After obtaining the login credentials, I was able to login successfully and grab the first flag:
<img width="957" height="123" alt="flag1" src="https://github.com/user-attachments/assets/c8dc1c4c-4f55-417b-a53b-bb59426d9fc7" />


## Escalation to root ⬆️
The final step was to find a way to escalate our privileges. Since we know the user's password, I ran `sudo -l` to see what the user can run with `sudo`:
```
sudo -l
```

<img width="953" height="261" alt="f" src="https://github.com/user-attachments/assets/18d2a601-83c8-4f8e-8323-276c7636d747" />

Now knowing that the user can run `/bin/tar` as the `root` user, I took advantage of this misconfiguration by referencing [GTFOBins](https://gtfobins.org/gtfobins/tar/#shell) to spawn a root shell using the tar binary:
```
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```
<img width="960" height="259" alt="root_flag" src="https://github.com/user-attachments/assets/254ec358-7677-48b9-a315-76c858823cac" />
- `tar` has the `--checkpoint` and `--checkpoint-action` flags that specify an action when a checkpoint is hit during archive processing. In this case it literally tells `tar` to execute a shell at that checkpoint so when `tar` runs as `root` via sudo, that shell inherits `root`


## Lessons Learned 📚
- **Enumerate every service, not just the web server.** The instinct is often to go straight for HTTP, but here the website was a complete dead end however the real foothold came from the FTP server. Checking each open port the `nmap` scan reported is what surfaced the anonymous FTP access.
- **Anonymous FTP is a feature, but exposed files are the flaw.** Anonymous login isn't a vulnerability by itself; the actual weakness was a **security misconfiguration** — sensitive files (`locks.txt` and `task.txt`) left readable in the anonymous-accessible directory. That exposure is what made everything downstream possible.
- **Read the "boring" files anyway.** `task.txt` looked like throwaway story flavor, but it was signed by `lin` handing over a likely username. Context clues like names, emails, and signatures are often deliberate breadcrumbs toward credentials.
- **Dead ends are part of the process.** Web enumeration on the site returned nothing useful, but rather than forcing that path, the right move was to pivot back to the data already in hand (the wordlist) and attack a different service.
- **A username plus a wordlist is a brute-force setup.** Pairing the `lin` username with `locks.txt` made `hydra` against SSH the obvious next step. Recognizing when you've been handed the two halves of a credential attack is a key enumeration skill.
- **Run `sudo -l` first for privilege escalation.** Once you have a valid password, checking what the user can run as root is the fastest, lowest effort privesc check and here it immediately revealed the `tar` misconfiguration.
- **GTFOBins is your first stop for binary abuse.** A binary runnable as root via `sudo` (like `tar`) is a classic, high-impact misconfiguration. 


## Tools/Resources Used 🧰
- `nmap`
- `hydra`
- `ffuf`
- [GTFOBins](https://gtfobins.org/)
