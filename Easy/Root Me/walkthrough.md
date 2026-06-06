# Root Me 💻
This TryHackMe room is a beginner-friendly challenge centered on web exploitation and privilege escalation. It begins with reconnaissance using tools like nmap to map out open ports and services, followed by directory enumeration to uncover a hidden file-upload page. From there, you bypass the upload filter to deliver a PHP reverse shell and gain an initial foothold on the machine. After capturing the user flag, the room walks you through privilege escalation by identifying a misconfigured SUID binary and abusing it to spawn a root shell, reinforcing core concepts like enumeration, filter evasion, and Linux privilege escalation along the way.


## Reconnaissance 🔍
Once connected to the TryHackMe network, my first order of business was to conduct a `nmap` scan to look for open ports on the machine to get an idea on what this machine is running:
```
nmap -sV -sC -p- -T4 10.64.143.151
```
<img width="935" height="627" alt="nmap_scan" src="https://github.com/user-attachments/assets/2702547d-36e3-46eb-a50e-044c2f209fe6" />

- Port 22 is open running `ssh` specifically version 8.2p1
- Port 80 is open running `http` specifically Apache 2.4.41
- The title of the website is **HackIT - Home**

The next step was to visit the website to see what I was working with:
<img width="960" height="1002" alt="image" src="https://github.com/user-attachments/assets/fa3ccf9a-be18-4e49-a7c6-726417f024f5" />

The main landing page for the website was bland and didn't have any clues on where to go next so using `ffuf`, I enumerated the website to find hidden webpages:
```
ffuf -u http://10.67.162.239/FUZZ -w /usr/share/wordlists/dirb/big.txt -e .js,.css,.txt,.php,.py,.html -mc 200,301,302
```
- The IP address is different in this command since the previous session timed out however the concept is the same. We're trying to find hidden directories/webpages in order to gain access to the machine

After enumerating the machine using `ffuf`, the following web pages were found:
<img width="1916" height="153" alt="webpages_ffuf" src="https://github.com/user-attachments/assets/bc4bcb74-f545-44f1-9e5d-4eed49098e5a" />


### Web Pages 📄
I then examined each directory directly to find a attack vector to gain access to the system. I first investigated the `css` directory:
<img width="954" height="1002" alt="css_directories" src="https://github.com/user-attachments/assets/72e39d24-4b09-4ffc-8923-bb10ae3973c6" />
- This directory contains two files `home.css` and `panel.css` which are the files that are responsible for the design and layout of the website not practicularly useful when it comes to finding a attack vector

The next one to look at was a webpage called `index.php`:
<img width="962" height="998" alt="index_php_ss" src="https://github.com/user-attachments/assets/f4fb2183-5d8b-4a65-9ea7-0070cd7df4a3" />
- Nothing of interest here, just the landing page for the website but it told me that this website was using `php` for their backend meaning it could be vulnerable to a **PHP reverse shell** attack

The next directory was `js`:
<img width="961" height="997" alt="js_ss" src="https://github.com/user-attachments/assets/0c1ef362-cddc-4d1b-b981-5c0356fe9acf" />
- Only one file was in here, `maquina_de_escrever.js` which translates to **typewriter.js** in English which is a Javascript plugin which gives text a cool typewriter effect which is displayed when you open the webpage

The next webpage I looked at was `panel`:
<img width="958" height="999" alt="panel_ss" src="https://github.com/user-attachments/assets/909bf36f-9097-40c2-9c33-2faa215ccdb3" />
- This webpage allows for file uploads which can be abused if the upload mechanism doesn't sanitize uploads which could allow the user to upload a malicious file to the server

The final webpage I looked at was `uploads`:
<img width="959" height="1005" alt="upload_ss" src="https://github.com/user-attachments/assets/90c4055e-830f-4f62-a9e6-88696fc524d0" />
- This directory is where all of the uploads are saved on the server

## Gaining Initial Access 🔓
After looking through all of the webpages/directories, the next step was to find a way to gain access to the system. Knowing we have an `upload` webpage, we can use this to our advantage and execute a **Unrestricted File Upload** attack. If the `upload` webpage doesn't validate a file's content, type, or extension an attacker can upload a **reverse shell** to the server which allows the attacker to execute **remote code execution** (arbitrary commands) onto the server which can lead to full system compromise. 

- A **reverse shell** acts like an SSH connection but the roles are reversed. Instead of the attacker initiating the connection, the target machine initiates a connection to the attacker over TCP/UDP. Using tools like `netcat` is crucial for this to work since it provides the TCP listener for the shell to connect to. Once connected, you're granted a basic `sh` terminal session, similar to a **SSH** session and execute commands. 
  
Based on my previous assumption that the website is using `php` for it's backend, I utilized [pentestmonkey's PHP reverse shell](https://pentestmonkey.net/tools/web-shells/php-reverse-shell) and attempted to upload it to the target machine:
- You can download the file directly off the website if click on the link or if you're using a Kali based machine you will be able to find it under `/usr/share/webshells/php`
- Make sure to setup a listener on a unused port so the reverse shell can connect back to your machine. In this case I used `nc`
- Make sure to edit the script to reflect your IP address
<img width="962" height="1000" alt="shell_upload" src="https://github.com/user-attachments/assets/e596e33c-484f-4cb1-8636-134a4edb1351" />


Upon uploading the file I ran into this error:
<img width="961" height="1002" alt="error" src="https://github.com/user-attachments/assets/bdc718a2-6625-4fde-8b74-4d2685def8d2" />
- The text translates to **PHP is not allowed!**

This could mean one of two things: 
- The upload mechanism blocks all files that contain `php`
- Or it only blocks files that have a `.php` extension

A quick Google search led me to this website (https://vulp3cula.gitbook.io/hackers-grimoire/exploitation/web-application/file-upload-bypass), which contains alternate `php` extensions:
<img width="783" height="123" alt="alternate_extensions" src="https://github.com/user-attachments/assets/745570ba-9b85-40cf-a7b3-bc59a114cb67" />


I went with `.php5` and attempted to upload the file again and it was a success:
<img width="957" height="1002" alt="success" src="https://github.com/user-attachments/assets/3a0c70ae-dc82-428d-8d3c-d6251bc07d14" />
<img width="960" height="1001" alt="upload_success" src="https://github.com/user-attachments/assets/3012aef4-4178-4d49-8aec-c3698d853e0e" />


The next step was to run the reverse shell. To do this I navigated to `uploads` directory and clicked on the filename which makes the webpage hang however this is normal execution. The server will then send a outbound connection back to your machine and you will be greeted with a `sh` terminal:
<img width="1902" height="331" alt="sh_session" src="https://github.com/user-attachments/assets/e7e220e5-c54b-4e12-8fb7-e9472eca9ee4" />

From there, I upgraded the reverse shell for better usability by running the following commands:
- Quick disclaimer, make sure the machine has `python` installed since this won't work if isn't on the machine
<img width="801" height="345" alt="shell_upgrade" src="https://github.com/user-attachments/assets/61459a7d-e33a-49e4-871f-614e8efe6983" />

After cleaning up the shell, I was able to find the 1st flag in `/var/www/user.txt`:
<img width="1283" height="190" alt="flag_1" src="https://github.com/user-attachments/assets/103f2b5a-8786-4b74-ae15-1aa99b4c5ec9" />

## Privilege Escalation
The final step was to find a way to elevate our privileges. In most cases, I usually check what the user can run with `sudo` by running `sudo -l` however we gained access by using a reverse shell which means we do not know the password of the user so that was the out of the question for me. 

The next course of action was to check for any file/binary that has the **SUID** bit set. The **SUID** bit allows a user to execute a file with the permissions of the file owner rather than the permissions of the user. If a file owned by the `root` user is misconfigured with the **SUID** bit, this can allow a underprivileged user to elevate their privileges as `root` and gain full system control. 

To check for files that have the **SUID** bit set I ran the following command:
```
find / -type f -perm -4000 2>/dev/null
```
<img width="783" height="839" alt="SUID" src="https://github.com/user-attachments/assets/2c6ec4e6-ad73-4f54-829a-23dd8e5e00c5" />
<img width="849" height="68" alt="python_ownedbyroot" src="https://github.com/user-attachments/assets/5bb8466c-7618-4a81-a4ed-205312ccd87b" />

The one file/binary that caught my attention was `/usr/bin/python2.7` which is owned by the `root` user so I checked if there was an exploit on (https://gtfobins.org/):
<img width="958" height="381" alt="exploit_for_python" src="https://github.com/user-attachments/assets/52b6893c-329f-449e-859c-84d8d57bca1c" />

Now all was left to do is run the command to elevate our privileges and find the root flag:
<img width="1024" height="745" alt="root_flag" src="https://github.com/user-attachments/assets/a0e1a488-0c48-4999-8484-4284ea21b5b5" />

## Lessons Learned 📝
- Enumerate broadly, not just for `.php`. Fuzzing with several extensions is what surfaced `panel` and `uploads` — the bland landing page gave nothing away, and a narrow wordlist or extension set would have missed the whole attack path.
- Fingerprint the backend early. Confirming the site ran on `php` directly informed the payload choice (a PHP reverse shell), saving time guessing at what would actually execute.
- Upload filters are often shallow. A "PHP is not allowed!" message rarely means all PHP is blocked — it usually blocks the `.php` extension specifically. Trying alternate extensions (`.php5`, `.phtml`, etc.) is a quick, high-value bypass to test.
- Understand how a reverse shell actually works. The roles are reversed: the target connects back to you, so a listener (`nc`) has to be running first, and the script's IP must point at your machine. The page "hanging" after triggering the shell is expected behavior, not a failure.
- Stabilize your shell before doing real work. Upgrading the raw `sh` session to a full TTY made enumeration and privesc far smoother — but it depends on the target having python, so it's worth confirming first.
- When `sudo -l` is off the table, pivot. A reverse shell gives you a session without the user's password, so the usual `sudo -l` check isn't available. SUID enumeration with find ... `-perm -4000` is the natural next move.
- SUID misconfigurations are a fast track to root. A single owner-root binary with the SUID bit set (here, Python) was enough to spawn a root shell and GTFOBins documents exactly how to abuse common binaries this way.

## Tools/Resources Used 🧰
- `nmap`
- `ffuf`
- `nc`
- [**pentester PHP reverse shell**](https://pentestmonkey.net/tools/web-shells/php-reverse-shell)
- [**GTFOBins**](https://gtfobins.org/)
- [**Hacker's Grimoire**](https://vulp3cula.gitbook.io/hackers-grimoire)
