# Agent Sudo -- TryHackMe Writeup

**Difficulty:** Easy  
**Category:** Web / Steganography / CVE-based Privilege Escalation

## Introduction
Agent Sudo is an easy-rated TryHackMe room that covers a wider range of techniques than most entry-level boxes, making it a solid step up from basic enumeration and privilege escalation. It begins with `nmap` reconnaissance, then requires manipulating the User-Agent header of an HTTP request to uncover a hidden agent codename. From there, `hydra` brute-forces FTP credentials that lead to a set of files containing hidden data, which steganography tools extract to reveal valid SSH credentials for an initial foothold. After the user flag, privilege escalation revolves around a known sudo vulnerability, CVE-2019-14287, that lets a restricted user run commands as root by manipulating the user ID.

### Quick Disclaimer
You will notice the IP addresses differ between commands. That is because I worked on the room across multiple sessions, and the IP changes each session.

## Enumeration
### Nmap Scan
The first step was an `nmap` scan to enumerate open ports and services:
```
nmap -sV -sC -T4 10.64.142.8
```
- `-sV` obtains the service version running on each port.
- `-sC` runs `nmap`'s default scripts for additional detail. On port 80, for example, the scripts pulled the `http-server-header` from the `Server:` field of the HTTP response header and the `http-title` from the `<title>` tag of the HTML body.
- `-T4` uses aggressive timing to speed up the scan without sacrificing reliability.

<img width="957" height="579" alt="Screenshot_2026-06-08_13-43-00" src="https://github.com/user-attachments/assets/4e7f71c8-dfa9-45f3-9dc1-0fb4302b120f" />

- Port 21 is running FTP, version vsftpd 3.0.3.
- Port 22 is running SSH, version 7.6p1.
- Port 80 is running an Apache web server, version 2.4.29.
- `nmap` also captured the page title: Announcement.

### Web Enumeration
Unlike some earlier rooms, anonymous login was disabled on port 21, so the next step was the website:
<img width="960" height="1001" alt="website_ss" src="https://github.com/user-attachments/assets/f183af42-a605-4e77-84ff-69de57778fa5" />

- The site instructs the visitor to use their own codename as the user agent to access the site.
- The User-Agent header identifies the browser (or client) making a network request. Nothing enforces its value, which makes it attacker-controlled: any HTTP client can set it to anything, and here the server uses it to gate access.

Using Burp Suite, I manually modified the `User-Agent` header to probe this behavior.

### Using Burp Suite
Opening Burp Suite, I started on the Target screen:
<img width="1920" height="1048" alt="burpsuite" src="https://github.com/user-attachments/assets/d76ac4b7-83c9-4afa-a85c-d8e6c81d65fa" />

- This screen opens a custom Chromium-based browser and builds a site map, an organized overview of the website.

Visiting the site through the Burp browser populated the request and response details:
<img width="1920" height="1044" alt="burpsuite_1" src="https://github.com/user-attachments/assets/3138751d-270b-43cc-a90e-4ea4b5a504ef" />

- On the left, the site's IP and the pages visited are shown, here only the root page, the site's top-level entry point.
- At the bottom, the HTTP request body, each header's value, and the HTTP response are displayed.

I sent the request to the Repeater module (right-click, "Send to Repeater"), modified the `User-Agent` header, and sent it:
<img width="1920" height="1045" alt="repeater_1" src="https://github.com/user-attachments/assets/37da63bf-9236-4d0a-9d45-53486cb790da" />

- A User-Agent of `C` worked. Since Agent R was already referenced on the landing page, the codenames appear to follow capital letters sequentially.
- Setting the User-Agent to `R` returns a response that further supports this:

<img width="1920" height="1044" alt="repeater" src="https://github.com/user-attachments/assets/db8f1252-812b-4ac2-9dbf-71cd9116ae74" />

Setting it to `C` redirected the request to `agent_C_attention.php`:
<img width="353" height="162" alt="hidden_page" src="https://github.com/user-attachments/assets/bc230e91-8e78-4a8a-8f05-54de71c2bd14" />
<img width="1920" height="927" alt="agent_c" src="https://github.com/user-attachments/assets/728c4017-ba34-4eac-95b2-00fa45e6d9bb" />

- Agent R addresses `chris` directly and notes that his password is weak, making `chris` a strong username candidate on the target machine.

## Gaining Access
### Background
With `chris` surfaced as a likely username, the next step was a credential brute-force. `hydra` tests each password in a wordlist against a login service until one succeeds. When the first targeted service does not yield a result, the same username and wordlist are worth trying against other authenticated services on the box, since credentials are often reused.

### Brute-Forcing Credentials
First against SSH:
```
hydra -l chris -P /usr/share/wordlists/rockyou.txt ssh://10.66.161.225
```
<img width="951" height="548" alt="hydra_attempt" src="https://github.com/user-attachments/assets/4c0cf72c-69e6-42ee-88cb-1abc3aab1f27" />

With no result on SSH, I pivoted to FTP using the same username and wordlist:
```
hydra -l chris -P /usr/share/wordlists/rockyou.txt ftp://10.66.161.225
```
<img width="939" height="560" alt="hydra_attempt2" src="https://github.com/user-attachments/assets/9eeac753-1881-454e-9c1b-c57cca387e9b" />

This succeeded, and I logged into the FTP server with the recovered credentials:
```
Username: chris
Password: crystal
ftp chris@10.66.161.225
```
<img width="920" height="282" alt="ftp_login" src="https://github.com/user-attachments/assets/89a24037-4bfb-4eb8-a623-0b9067b85d06" />

Listing the contents revealed three files:
<img width="954" height="223" alt="ftp_files" src="https://github.com/user-attachments/assets/cfca7529-e32e-4fca-a9f4-570254c2f707" />

After downloading them, the first file I read was `To_agentJ.txt`:
<img width="1920" height="228" alt="agentJ" src="https://github.com/user-attachments/assets/af79f9ad-a572-4c73-a92b-63669e4af00b" />

- Agent C tells Agent J that their login credentials are stored inside an image in their directory, meaning one of the two retrieved image files likely contained hidden data.

### How the Exploit Works: Steganography
Steganography conceals data inside another file, commonly an image, without changing its visible appearance. The embedded payload is usually protected by a passphrase, and if that passphrase is weak it can be recovered with a wordlist. The note in `To_agentJ.txt` was the tell that credentials were buried inside one of the images rather than sitting in plain sight.

Using `stegseek`, I cracked the passphrase and extracted the hidden message:
<img width="1295" height="515" alt="stegseek_crack" src="https://github.com/user-attachments/assets/3d3ab138-30de-4cbe-a8d5-499fbfd80eb4" />

The extracted data gave a valid set of SSH credentials:
```
Username: james
Password: hackerrules!
```

Logging in as `james`, I captured the user flag:
<img width="963" height="861" alt="user_flag" src="https://github.com/user-attachments/assets/f6797e7e-bc28-4153-89fd-92aec9413682" />

## Privilege Escalation
### How the Exploit Works: CVE-2019-14287
The first check was `james`'s sudo permissions:
```
sudo -l
```
<img width="1919" height="173" alt="sudo_l" src="https://github.com/user-attachments/assets/10298e49-5f3f-47a7-9f90-90cc2566ea1a" />

- `james` can run `/bin/bash` with sudo as any user except root.

That "except root" restriction looks like a dead end, and a check for SUID binaries came up empty too:
```
find / -type f -perm -4000 2>/dev/null
```
<img width="726" height="914" alt="perm_4000" src="https://github.com/user-attachments/assets/83817501-2480-4aaa-939d-9e9f4ec0fa73" />

- Cross-referencing the results against [GTFOBins](https://gtfobins.github.io/), none offered a viable path.

Researching the exact sudo permission string surfaced **CVE-2019-14287**. The restriction is enforced by username, but sudo's user-ID handling has a flaw: passing `-u#-1` requests user ID -1, which the system wraps around to user ID 0, root. Because the policy only blocks the root *name* and never validates the numeric ID, `sudo -u#-1` runs the command as root while slipping past the "except root" rule entirely. The full exploit details are on [Exploit-DB](https://www.exploit-db.com/exploits/47502).

### Exploiting CVE-2019-14287
Running the exploit returned a root shell and the root flag:
<img width="1919" height="465" alt="root_flag" src="https://github.com/user-attachments/assets/0be85f6b-9382-4b40-a20d-ecb5529d33af" />

## Flag Capture
Both flags were recovered: the user flag after logging in as `james` with the SSH credentials recovered via steganography, and the root flag after exploiting CVE-2019-14287 (`sudo -u#-1`) to bypass the "except root" restriction and spawn a root shell.

## Lessons Learned
- **Not all FTP servers allow anonymous login, so enumerate before assuming.** Earlier rooms like Bounty Hacker had anonymous FTP wide open, but here it was locked down. Trying it and moving on quickly is the right habit.
- **Read the webpage carefully before reaching for tools.** The landing page told us exactly what to do: change the User-Agent header. Recognizing that clue saved time that would have gone to directory fuzzing.
- **User-Agent headers are attacker-controlled.** Browsers set this header automatically, but nothing enforces it; any HTTP client can set it to anything. Using header values to gate access is a fragile, easily bypassed control.
- **Try adjacent services when one brute-force fails.** `hydra` against SSH went nowhere, but the same username and wordlist against FTP worked immediately. Credentials that fail on one service may still be valid elsewhere on the same host.
- **Hidden data does not have to look hidden.** The FTP files looked like ordinary images and a text file. `To_agentJ.txt` was the tell that pointed at steganography rather than guesswork.
- **stegseek is much faster than steghide for cracking passphrases.** It handles wordlist-based passphrase cracking against steganographic content natively and quickly, the right tool for this job.
- **sudo -l output deserves more than a surface read.** "Any user except root" looks like a dead end, and SUID enumeration came up empty. Researching the exact permission string is what surfaced CVE-2019-14287, a case where the vulnerability lives in the policy wording rather than a misconfigured binary.
- **CVE-2019-14287 works because sudo does not validate the user ID.** `-u#-1` resolves to user ID 0 (root). The "except root" rule checks only the name, not the numeric ID, a subtle implementation flaw that bypasses a well-intentioned policy.

## Tools Used
- `nmap` — Reconnaissance with `-sV -sC -T4` to enumerate open ports, service versions, and the HTTP page title via default scripts.
- Burp Suite — Mapped the site with the Target module and modified the `User-Agent` header with the Repeater module to reveal the hidden agent page.
- `hydra` — Credential brute-forcing, first against SSH, then pivoted to FTP using `chris` and `rockyou.txt`.
- `ftp` — Authenticated access to the FTP server to retrieve the hidden-data files.
- `stegseek` — Wordlist-based passphrase cracking against the steganographic image to extract the SSH credentials.
- `find` — SUID binary enumeration via `find / -type f -perm -4000 2>/dev/null`.
- GTFOBins — Cross-referenced the SUID list for known exploitation methods (none matched here).
- CVE-2019-14287 (`sudo -u#-1`) — Exploited the flaw in sudo's user-ID validation to bypass the "except root" restriction.
- Exploit-DB — Source for the CVE-2019-14287 exploit details.
