# Agent Sudo 👽
This TryHackMe room is a beginner-friendly challenge that covers a wider range of techniques than most entry-level rooms, making it a solid step up from basic enumeration and privesc. It begins with `nmap` reconnaissance to identify open ports and services, followed by manipulating the **User-Agent** header in an HTTP request to uncover a hidden agent codename. From there, `hydra` is used to brute-force FTP credentials, which surfaces a set of files containing hidden data. Extracting that data requires **steganography** tools to uncover concealed information buried inside the files, eventually yielding valid SSH credentials for an initial foothold. After capturing the user flag, privilege escalation revolves around a known `sudo` vulnerability (**CVE-2019-14287**) that allows a restricted user to run commands as root by manipulating the user ID — reinforcing a diverse set of skills including service enumeration, HTTP header manipulation, credential brute-forcing, steganography, and CVE-based privilege escalation along the way.

### Quick Disclaimer
You'll notice that the IP addresses are different in different commands and that's because I worked on the challenge in multiple sessions and in each session, the IP address changes.


## Reconnaissance 🔍
The first step was to run a `nmap` scan to enumerate the machine and find open ports and services running on the machine:
```
nmap -sV -sC -T4 10.64.142.8
```
- `-sV` obtains the service version of the service
- `-sC` runs default `nmap` scripts on services found and obtains additional information; for example on port 80 the script pulled the `http-server-header` from the `Server:` field in the HTTP response header and the `http-title` from the `<title>` tag in the HTML response body
- `-T4` uses aggressive timing to speed up the scan without sacrificing reliability

<img width="957" height="579" alt="Screenshot_2026-06-08_13-43-00" src="https://github.com/user-attachments/assets/4e7f71c8-dfa9-45f3-9dc1-0fb4302b120f" />

- port 21 is running the `ftp` service specifically version `vsftpd 3.0.3`
- port 22 is running `ssh` specifically version `7.6p1`
- port 80 is running an Apache web server specifically **Apache 2.4.29**
- `nmap` also picked up the title of the webpage being **Announcement**

Unlike previous rooms, anonymous login was disabled on port 21 so the next logical step was to check the website:
<img width="960" height="1001" alt="website_ss" src="https://github.com/user-attachments/assets/f183af42-a605-4e77-84ff-69de57778fa5" />
- The website states to use our own **codename** as the user agent to access the website
- The **user agent** header specifies what browser we are using to make a network request

Using **Burpsuite**, I manually edited the `User-Agent` header to access the website.


### Burpsuite
Upon opening **Burpsuite**, I was greeted with this screen:
<img width="1920" height="1048" alt="burpsuite" src="https://github.com/user-attachments/assets/d76ac4b7-83c9-4afa-a85c-d8e6c81d65fa" />
- This is the **Target** screen where it opens a custom Chromium based web browser so it can build a site map showing an organized overview of the website

Once I visited the site on the Burpsuite browser, the following information showed up:
<img width="1920" height="1044" alt="burpsuite_1" src="https://github.com/user-attachments/assets/3138751d-270b-43cc-a90e-4ea4b5a504ef" />

- On the left hand side the IP address of the website is shown along with the pages visited; in this case only the **root page** which is the top level entry point of the website
- Towards the bottom of the screen the body of the HTTP request is shown along with the values for each header and the HTTP response

The next step was to edit the HTTP request and change the **User-Agent** header using the **Repeater** module:
- Before modifying the request I sent the original HTTP request to the Repeater module by right clicking on the request and clicking **Send to Repeater**

Once the request was loaded into the Repeater module I modified the **User-Agent** header and sent the request:
<img width="1920" height="1045" alt="repeater_1" src="https://github.com/user-attachments/assets/37da63bf-9236-4d0a-9d45-53486cb790da" />

- **C** worked as the User-Agent based on the nature of this challenge — since **Agent R** was already referenced on the landing page, the codenames appear to follow capital letters sequentially
- Changing the User-Agent to **R** returns the following response, which further supports this:

<img width="1920" height="1044" alt="repeater" src="https://github.com/user-attachments/assets/db8f1252-812b-4ac2-9dbf-71cd9116ae74" />

Changing the User-Agent redirected the request to `agent_C_attention.php`:
<img width="353" height="162" alt="hidden_page" src="https://github.com/user-attachments/assets/bc230e91-8e78-4a8a-8f05-54de71c2bd14" />
<img width="1920" height="927" alt="agent_c" src="https://github.com/user-attachments/assets/728c4017-ba34-4eac-95b2-00fa45e6d9bb" />
- **Agent R** addresses **chris** directly and mentions that his password is weak — making `chris` a strong candidate for a username on the target machine

## Brute Forcing 🔓
`chris` was a likely username on the target machine so I attempted to brute force his `ssh` password using `hydra`:
```
hydra -l chris -P /usr/share/wordlists/rockyou.txt ssh://10.66.161.225
```
<img width="951" height="548" alt="hydra_attempt" src="https://github.com/user-attachments/assets/4c0cf72c-69e6-42ee-88cb-1abc3aab1f27" />

After the SSH brute-force returned no results, I pivoted to targeting the FTP service instead using the same credentials:
```
hydra -l chris -P /usr/share/wordlists/rockyou.txt ftp://10.66.161.225
```
<img width="939" height="560" alt="hydra_attempt2" src="https://github.com/user-attachments/assets/9eeac753-1881-454e-9c1b-c57cca387e9b" />

After a successful brute-force, I logged into the `ftp` server using the recovered credentials:
```
Username: chris
Password: crystal
ftp chris@10.66.161.225
```
<img width="920" height="282" alt="ftp_login" src="https://github.com/user-attachments/assets/89a24037-4bfb-4eb8-a623-0b9067b85d06" />

Once inside the `ftp` server I listed the contents and found 3 files:
<img width="954" height="223" alt="ftp_files" src="https://github.com/user-attachments/assets/cfca7529-e32e-4fca-a9f4-570254c2f707" />

After downloading them to my working directory, the first file I investigated was `To_agentJ.txt`:
<img width="1920" height="228" alt="agentJ" src="https://github.com/user-attachments/assets/af79f9ad-a572-4c73-a92b-63669e4af00b" />
- Agent C tells Agent J that their login credentials are stored inside an image within their directory — meaning one of the two image files retrieved from the FTP server likely contained hidden data

Using `stegseek`, I cracked the passphrase of the image and extracted the hidden message:
<img width="1295" height="515" alt="stegseek_crack" src="https://github.com/user-attachments/assets/3d3ab138-30de-4cbe-a8d5-499fbfd80eb4" />

With the extracted data I now had a valid set of SSH credentials:
```
Username: james
Password: hackerrules!
```

After logging in I captured the user flag:
<img width="963" height="861" alt="user_flag" src="https://github.com/user-attachments/assets/f6797e7e-bc28-4153-89fd-92aec9413682" />


## Privilege Escalation ⬆️
The final step was to escalate privileges. The first check was to review the user's `sudo` permissions:
```
sudo -l
```
<img width="1919" height="173" alt="sudo_l" src="https://github.com/user-attachments/assets/10298e49-5f3f-47a7-9f90-90cc2566ea1a" />
- The user **james** can run `/bin/bash` with `sudo` as any user except the root user

The next step was to check for misconfigured binaries with their SUID bit set:
```
find / -type f -perm -4000 2>/dev/null
```
<img width="726" height="914" alt="perm_4000" src="https://github.com/user-attachments/assets/83817501-2480-4aaa-939d-9e9f4ec0fa73" />
- After cross-referencing the results against [GTFOBins](https://gtfobins.github.io/), none of the binaries returned a viable exploitation path

Returning to the `sudo -l` output, researching the exact permission string surfaced **CVE-2019-14287** — a flaw in sudo's user ID validation where passing `-u#-1` causes sudo to resolve the user ID to 0 (root), bypassing the "except root" restriction entirely. The full exploit details can be found on [Exploit-DB](https://www.exploit-db.com/exploits/47502).

Executing the exploit returned a root shell and the root flag:
<img width="1919" height="465" alt="root_flag" src="https://github.com/user-attachments/assets/0be85f6b-9382-4b40-a20d-ecb5529d33af" />

## Lessons Learned 📚

- **Not all FTP servers allow anonymous login — enumerate before assuming.** Previous rooms like Bounty Hacker had anonymous FTP wide open, but here it was locked down. Trying it and moving on quickly is the right habit rather than forcing a path that isn't there.

- **Read the webpage carefully before reaching for tools.** The landing page directly told us what to do — change the User-Agent header. The clue was sitting in plain text; recognizing it saved time that would have been wasted on directory fuzzing.

- **User-Agent headers are attacker-controlled.** Browsers set this header automatically, but nothing enforces it — any HTTP client (Burpsuite Repeater, `curl`) lets you set it to whatever you want. This room is a good demonstration that servers sometimes use header values to gate access, which is a fragile and easily bypassed control.

- **Try brute-forcing adjacent services when one fails.** `hydra` against SSH with `rockyou.txt` went nowhere, but pivoting to FTP with the same username and wordlist worked immediately. When one service doesn't bite, the credentials may still be valid elsewhere on the same machine.

- **Hidden data doesn't have to look hidden.** The FTP files looked like ordinary images and a text file. `To_agentJ.txt` was the tell — it explicitly said credentials were buried inside an image. Reading every file before reaching for tools is what pointed at steganography rather than guessing at it.

- **`stegseek` is significantly faster than `steghide` for cracking passphrases.** Rather than manually guessing or writing a loop, `stegseek` handles wordlist-based passphrase cracking against steganographic content natively and quickly — the right tool for this specific job.

- **`sudo -l` output deserves more than a surface read.** The restriction "any user except root" looks like a dead end, and SUID enumeration came up empty too. Going back and researching the exact sudo permission string is what surfaced **CVE-2019-14287** — a case where the vulnerability lives in the *policy wording* rather than a misconfigured binary.

- **CVE-2019-14287 works because `sudo` doesn't validate the user ID.** Passing `-u#-1` tells sudo to run as user ID -1, which the kernel wraps around to UID 0 (root). The restriction "except root" only checks by name, not by the numeric ID — a subtle implementation flaw that bypasses what looks like a well-intentioned policy.

## Tools Used 🛠️

- **`nmap`** — Initial reconnaissance with `-sV -sC -T4` to enumerate open ports, service versions, and pull the HTTP page title via default NSE scripts.
- **Burpsuite** — Used the Target module to map the site and the Repeater module to manually modify the `User-Agent` HTTP request header to reveal the hidden agent page.
- **`hydra`** — Credential brute-forcing; first attempted against SSH, then pivoted to FTP using `chris` as the username and `rockyou.txt` as the wordlist.
- **`ftp`** — Authenticated access to the FTP server using the recovered credentials to retrieve `cutie.png`, `cute-alien.jpg`, and `To_agentJ.txt`.
- **`stegseek`** — Wordlist-based passphrase cracking against the steganographic image to extract the hidden SSH credentials.
- **`find`** — SUID binary enumeration via `find / -type f -perm -4000 2>/dev/null` to search for privilege escalation paths.
- **GTFOBins** — Cross-referenced the SUID binary list for known exploitation methods (none matched in this case).
- **CVE-2019-14287 (`sudo -u#-1`)** — Exploited a flaw in sudo's user ID validation to bypass the "except root" restriction and spawn a root shell.
- **Exploit-DB** — Source for the CVE-2019-14287 exploit details and usage.
