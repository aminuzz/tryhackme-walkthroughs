# Brooklyn Nine Nine 🔍
This challenge covers the fundamentals of penetration testing and follows a step-by-step process from enumeration to privilege escalation. 
It begins with identifying exposed services such as FTP, then progresses into techniques like image steganography and 
gaining SSH access to the system. The challenge gradually increases in depth by requiring users to explore the machine 
for misconfigurations, ultimately leading to privilege escalation through a lesser-known Nano editor technique.

## User flag 🚩
I first ran `nmap` to look for open/vulnerable ports on the machine:
<img width="1318" height="928" alt="image" src="https://github.com/user-attachments/assets/9bad6689-cd50-4d2c-88aa-81e4cf0973df" />
- Port 21 is open and allows for `anonymous ftp` login meaning we can login as the `anonymous` user without needing a password.
  - Within the `ftp` server there is a file called `note_to_jake.txt` (something of interest).
- Port 22 is open for `ssh` remote login.
- Port 80 is open which is hosting a `Apache` web server meaning this machine is hosting a website that we can take a look at.

Before we dive deeper into the `ftp` server I checked the website to see what I was working with:
<img width="1920" height="1044" alt="website_screenshot" src="https://github.com/user-attachments/assets/9bab2814-180f-4d20-a851-b94709628db3" />

The site is bare bones and there isn't much going on so I went ahead and checked the source code of the site to find more clues:
<img width="1919" height="925" alt="source_code" src="https://github.com/user-attachments/assets/0cea67aa-2800-4062-8995-1606f56aa33c" />
- The source code gives us a clue that this background image might be using steganography to hide data
- The file path of the image is also given away `brooklyn99.jpg`

The next step was to download the image and extract the hidden data:
<img width="1026" height="258" alt="image" src="https://github.com/user-attachments/assets/d624607b-d705-4929-80d0-2a680ed55411" />
- We can use a tool like `stegseek` which allows us to crack the passphrase of the image so we can read what's hidden inside the image
- This tool by default uses `rockyou.txt` as the wordlist for this attack

<img width="934" height="251" alt="Screenshot_2026-05-21_09-59-19" src="https://github.com/user-attachments/assets/a5bfc3c7-15a6-4f52-9500-3830000af945" />

The output gives us the following:
- The passphrase that was used to protect the data, in this case it was `admin`
- It extracted the original filename `note.txt`
- The filename with the extracted data `brooklyn99.jpg.out`

Let's look into what the outputted file contains:
<img width="917" height="201" alt="holt_passwd" src="https://github.com/user-attachments/assets/9454de7c-2691-4960-8088-fd2f6613f98b" />
- We are given the password for the user `holt` which will allow us to access their machine

Before we login as `holt` I checked out the `ftp` server which contained the file I mentioned earlier:
<img width="1920" height="659" alt="ftp" src="https://github.com/user-attachments/assets/b02eec4f-868f-4183-8b21-eecaf6a60640" />
<img width="1920" height="259" alt="Screenshot_2026-05-21_10-16-35" src="https://github.com/user-attachments/assets/951c63eb-cfd2-4de4-a7bb-5b9c9051542c" />
- Now we have another user, `jake`, to look out for

After that I went ahead and logged in as the user `holt` on the machine and luckily the flag was in the home directory:
<img width="604" height="143" alt="user_flag" src="https://github.com/user-attachments/assets/08646d20-8063-481b-9757-8c5989069eea" />

## Root flag 🚩
In order to escalate our privileges to the root user, we need to find a misconfiguration or executable with elevated permissions that can be used to gain higher level access. The first thing that came into my mind was to check the `sudoers` file:
- This file allows us to see what commands a user can run with `sudo`
<img width="1832" height="200" alt="sudoer_file" src="https://github.com/user-attachments/assets/3b0e84fc-f021-41dd-bc6b-8d09f429449a" />

- In this case the user `holt` can run `/bin/nano` with `sudo` as any user without a password. This is our privilege escalation vector to gain root access on the machine.

I used [gtfobins.org](https://gtfobins.org/) as my reference to execute the privilege escalation technique specifically the page about `nano` shown below:
<img width="1280" height="825" alt="nano" src="https://github.com/user-attachments/assets/26289ce9-b3c6-44a8-afa9-be3622fef1b5" />

Following the directions I was able to obtain a root shell and from there I was able to get the root flag:
<img width="850" height="377" alt="root_flag" src="https://github.com/user-attachments/assets/a9c445e9-c3d5-457f-bb6e-cc858a46ad1b" />


## Lessons learned 📘
- Always begin with proper enumeration since open ports and exposed services can quickly reveal an attack path.
- Anonymous FTP login can expose sensitive files or usernames that may help with gaining access.
- Checking the source code of a website can reveal hidden clues, comments, or file paths that are not immediately visible on the webpage itself.
- Steganography can be used to hide sensitive information inside image files, making tools like stegseek useful during investigations.
- Weak passphrases are vulnerable to dictionary attacks using common wordlists such as rockyou.txt.
- mMisconfigured sudo permissions can lead directly to privilege escalation if dangerous binaries are allowed to run as root.
- GTFOBins is an extremely useful resource for identifying ways to abuse legitimate Linux binaries for privilege escalation.
- Always check what commands a user can execute with sudo by running sudo -l.



## Tools 🧰
- `nmap` — Port scanning and service enumeration
- `ftp` — Accessing the anonymous FTP server
- `wget` — Downloading files from the target website
- `stegseek` — Cracking steganography passphrases and extracting hidden files
- `ssh` — Remote login to the target machine
- `sudo` — Checking elevated command permissions
- `nano` — Exploited for privilege escalation
- **GTFOBins** — Reference for Linux binary privilege escalation techniques
