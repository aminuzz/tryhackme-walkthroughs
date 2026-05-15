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


