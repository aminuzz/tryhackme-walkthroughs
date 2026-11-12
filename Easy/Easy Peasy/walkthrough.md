# Easy Peasy -- TryHackMe Writeup

**Difficulty:** Easy
**Category:** Web Enumeration / Encoding / Steganography / Cron Privilege Escalation

## Introduction
Easy Peasy is an easy-rated TryHackMe room that chains together a long sequence of small techniques. It uses `nmap` and `gobuster` to locate hidden content across two web servers, works through several layers of encoding and hash cracking to recover SSH credentials, uses steganography to extract a hidden password, and finishes with privilege escalation through a vulnerable cron job. The room rewards patient enumeration and a familiarity with common encoding and cracking tools.

## Task 1: Nmap Enumeration
The first few questions concern the open ports and services.

### Question 1: How many ports are open?
A full TCP port scan:
```
nmap -sV -p1-65535 10.66.143.198
```
- `-sV` returns the service and version running on each port.
- `-p1-65535` scans all ports. By default `nmap` scans the top 1000, but a thorough scan of every port is the right choice here.
![Gif16](https://github.com/user-attachments/assets/bf20b90c-5652-4dee-b968-9e57a5169d60)

- In total, 3 ports were open.
<img width="836" height="68" alt="image" src="https://github.com/user-attachments/assets/3d3ce26a-298c-4e58-a3c4-8caae2fa6107" />

### Question 2: What is the version of nginx?
<img width="1464" height="901" alt="image" src="https://github.com/user-attachments/assets/11816e88-d96a-4297-b8d6-85578fa13301" />

Because the scan used `-sV`, it returned service versions. On port 80, an nginx web server was running version 1.16.1.
<img width="845" height="71" alt="image" src="https://github.com/user-attachments/assets/706e0689-56a9-4757-ae7c-d24d58acc198" />

### Question 3: What is running on the highest port?
<img width="1475" height="914" alt="image" src="https://github.com/user-attachments/assets/9ed07283-70b3-4efa-a660-27439eb497d8" />

An Apache web server is running on port 65524, the highest open port.
<img width="836" height="64" alt="image" src="https://github.com/user-attachments/assets/18ffadb9-334e-4067-a7c8-ab913f1e8040" />

## Task 2: Gobuster Enumeration
The next questions use `gobuster` to enumerate the nginx and Apache web servers.

### Question 1: Find flag 1
I started with the nginx server, since that was the first thing visible in the browser:
<img width="1476" height="915" alt="image" src="https://github.com/user-attachments/assets/4db5ca87-c5c1-40d3-aadd-82f4ccbaec6a" />

Running `gobuster` in directory mode:
```
gobuster dir -u "http://10.67.190.178" -w /usr/share/wordlists/dirb/big.txt -t 64
```
![Gif17](https://github.com/user-attachments/assets/7655bb6c-5117-4ae8-b2fb-4648ad31962d)

- Gobuster located two hidden pages: `hidden` and `robots.txt`.

`robots.txt` is a file placed in a site's root directory that tells search engine crawlers which pages to crawl or ignore:
<img width="1472" height="910" alt="image" src="https://github.com/user-attachments/assets/14e2bca0-a1b4-4e9b-89f1-a7e3ba7f7e50" />

What it means:
- `User-Agent: *` applies to all crawlers.
- `Disallow: /` tells crawlers not to access anything on the site.
- `Robots Not Allowed` is just a comment.

Looking at `http://10.67.190.178/hidden/`:
<img width="1476" height="916" alt="image" src="https://github.com/user-attachments/assets/b6bceac0-085f-4644-a7a3-d353616b37b4" />

And its HTML source:
<img width="1452" height="910" alt="image" src="https://github.com/user-attachments/assets/25f875f9-393b-480a-877a-9061575cd946" />

After reviewing the HTML, I enumerated further under `http://10.67.190.178/hidden/`:
![Gif18](https://github.com/user-attachments/assets/931047bf-6508-4570-bef2-a86103aec9ff)

- There is a subdirectory within `/hidden` called `whatever`, worth investigating in the browser.
<img width="1475" height="922" alt="image" src="https://github.com/user-attachments/assets/f67febeb-f414-4a11-9761-7fc4619b5636" />

Nothing visible on the page, so I checked its source:
<img width="1470" height="921" alt="image" src="https://github.com/user-attachments/assets/9e7874ed-150b-4710-9f5b-03b31c187277" />

- There is a hidden paragraph element containing a Base64 string.

Decoding the string with CyberChef:
<img width="1919" height="938" alt="image" src="https://github.com/user-attachments/assets/1a2fd39b-8a62-40a0-b915-33a36a5b5d7e" />

- The resulting text is the answer to Question 1.
<img width="841" height="66" alt="image" src="https://github.com/user-attachments/assets/4f421237-a5d4-49c0-bc5e-5f7a6a0f661d" />

### Question 2: What is flag 2?
Based on the earlier scan, an Apache web server was also running on port 65524, so that became the next target:
```
gobuster dir -u "http://10.65.188.233:65524" -w /usr/share/wordlists/dirb/common.txt -t 64
```
![Gif19](https://github.com/user-attachments/assets/6427909b-0542-4691-a330-8bacd42de968)

Gobuster found `index.html` and `robots.txt`. Investigating `robots.txt`, I noticed an MD5 hash in the User-Agent section, which I decrypted with [hashes.com](https://hashes.com/en/decrypt/hash) to recover the second flag:
<img width="1373" height="915" alt="image" src="https://github.com/user-attachments/assets/29de7362-b26c-438c-b933-3f4a27bc7f61" />
<img width="843" height="71" alt="image" src="https://github.com/user-attachments/assets/fc332d23-3a58-4108-b0b7-1c80e752418a" />

### Question 3: Crack the hash with easypeasy.txt. What is flag 3?
Looking through the source code on the Apache root web page, there is a string containing the answer:
<img width="1479" height="919" alt="image" src="https://github.com/user-attachments/assets/b39fb2e3-6df7-4ec5-bfcf-d9e60a2414c2" />

### Question 4: What is the hidden directory?
<img width="1479" height="918" alt="image" src="https://github.com/user-attachments/assets/21b4f6eb-5e7c-4c57-96bc-6f4f55b59572" />

In the highlighted section, there is a hidden paragraph element with the text:
- `its encoded with ba....:ObsJmP173N2X6dOrAgEAL0Vu`

After some trial and error in CyberChef, I decoded the text from Base62 to plaintext, revealing a hidden directory called `/n0th1ng3ls3m4tt3r`.
<img width="842" height="77" alt="image" src="https://github.com/user-attachments/assets/42fa7b13-0ce8-4b5f-bccf-6b28bc938ca7" />

### Question 5: Crack the hash with the provided wordlist. What is the password?
Visiting the hidden directory, I found a hash buried in the page source:
<img width="1475" height="918" alt="image" src="https://github.com/user-attachments/assets/26774697-72a2-449d-9f54-01c234ed170b" />

- The room hints that this is a GOST hash.

Downloading the provided wordlist and saving the hash to a file, John the Ripper cracks it:
<img width="967" height="254" alt="image" src="https://github.com/user-attachments/assets/80d9b6ea-1a4e-4ea5-8682-be880690ff5f" />

The recovered password is `mypasswordforthatjob`:
<img width="837" height="82" alt="image" src="https://github.com/user-attachments/assets/2018b18b-8d1d-4c60-b200-5895b70780a8" />

### Question 6: What is the password to log in via SSH?
### How the Exploit Works: Steganography
The previous question recovered a hash from the page source, so my next assumption was that the SSH password was hidden the same way, in one of the images on the site. Steganography conceals data inside another file (here, an image) without changing its visible appearance; the embedded payload is often protected by a passphrase. If that passphrase is known, the hidden data can be extracted.
<img width="1463" height="924" alt="image" src="https://github.com/user-attachments/assets/8598b40d-5166-42a9-9364-aa90db66f35e" />

- The earlier hash lived in the background image, so I focused on the image containing binary numbers and saved it for investigation.
<img width="1471" height="905" alt="image" src="https://github.com/user-attachments/assets/6a30fe4c-fb0d-4bb0-b592-fd58f5870590" />

Using `steghide`, I extracted the hidden data. The extraction passphrase was the password cracked in the previous question:
![Gif20](https://github.com/user-attachments/assets/0ec37ccd-e636-4c96-9459-f5e75050d0c8)
- The result was written to `secrettext.txt`, containing a username, `boring`, and a password in binary format.

Converting the binary back to plaintext with CyberChef:
<img width="950" height="936" alt="image" src="https://github.com/user-attachments/assets/e7e4ceda-feba-4e4e-96ab-18f9c0cef1ed" />
<img width="834" height="77" alt="image" src="https://github.com/user-attachments/assets/04fb9969-24e3-4c2f-9877-f208a0f8de68" />

### Question 7: What is the user flag?
Logging in over SSH with the recovered credentials, the flag was in the home directory. The text was encoded with the ROT13 cipher, so CyberChef decoded it:
<img width="807" height="432" alt="image" src="https://github.com/user-attachments/assets/132bd068-cc99-4a2b-86fb-7965d2fb844a" />
<img width="958" height="945" alt="image" src="https://github.com/user-attachments/assets/ab1aa9d2-8d0f-4d48-8094-37113f52a2b6" />
<img width="844" height="77" alt="image" src="https://github.com/user-attachments/assets/03fa2171-f84d-40f4-b8ce-eddf3cf9a60a" />

### Question 8: What is the root flag?
### How the Exploit Works: Writable Root Cron Job
The room's description points to escalation through a vulnerable cron job. A cron job is a task the operating system runs automatically at a set time (clearing folders, running backups, and so on), and jobs can be system-wide or user-specific. The vulnerability arises when a cron job that runs as root executes a script that a lower-privileged user can modify. Because the script's contents run with root's privileges on the next scheduled execution, an attacker who can edit that script can have it run any command as root, such as a reverse shell.

Reviewing the system-wide cron jobs in `/etc/crontab`, one stood out:
<img width="958" height="945" alt="image" src="https://github.com/user-attachments/assets/f8647505-139e-4482-9667-1ecca717f2af" />
<img width="543" height="73" alt="image" src="https://github.com/user-attachments/assets/b1be9043-1fae-4f8f-9135-4d73546512a9" />

Anything in this cron job runs as root, so I inserted a reverse shell into `.mysecretcronjob.sh` and set up a `netcat` listener to catch an interactive root shell:
<img width="936" height="706" alt="image" src="https://github.com/user-attachments/assets/6d905006-886c-4aea-8c82-048f55b9dbde" />

Once the listener received the connection on port 1000, I confirmed I was root and found the flag in `.root.txt`:
<img width="623" height="483" alt="image" src="https://github.com/user-attachments/assets/60262ee5-6c4e-4670-ab68-b03dab774776" />
<img width="848" height="88" alt="image" src="https://github.com/user-attachments/assets/2a19b76f-58c7-4f6d-849f-e9390b896a45" />

## Flag Capture
Both the user and root flags were recovered: the user flag from `boring`'s home directory (ROT13-decoded) after chaining web enumeration, hash cracking, and steganography into SSH credentials, and the root flag from `.root.txt` after abusing a writable root cron job (`.mysecretcronjob.sh`) to catch a root reverse shell.

## Lessons Learned
- **Thorough enumeration is critical.** A full port scan with `nmap -p1-65535` identified a service running on a non-standard port (Apache on 65524) that a default scan would have missed.
- **Web enumeration reveals hidden content.** Gobuster uncovered hidden paths like `/hidden` and `/whatever`, which led to encoded data and further clues.
- **robots.txt is reconnaissance gold.** Though meant for crawlers, it exposed hidden paths and hints worth investigating.
- **Source code inspection matters.** Viewing HTML source revealed hidden elements and encoded strings not visible on the rendered page.
- **Know common encoding schemes.** Base64, Base62, ROT13, and binary-to-text conversion were all needed to uncover flags throughout the room.
- **Password cracking tools are essential.** John the Ripper with the provided wordlist cracked the GOST hash successfully.
- **Steganography hides data in images.** Tools like `steghide` extract embedded data when the correct passphrase is supplied, here, a password cracked in an earlier step.
- **Writable root cron jobs are a direct path to root.** A cron job running a script that a lower-privileged user can edit is enough to execute arbitrary commands as root.

## Tools Used
- Nmap
- Gobuster
- CyberChef
- John the Ripper
- Steghide
- Netcat
