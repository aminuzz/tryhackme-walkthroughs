# Brooklyn Nine Nine 🔍
This challenge covers the fundamentals of penetration testing and follows a step-by-step process from enumeration to privilege escalation. 
It begins with identifying exposed services such as FTP, then progresses into techniques like image steganography and 
gaining SSH access to the system. The challenge gradually increases in depth by requiring users to explore the machine 
for misconfigurations, ultimately leading to privilege escalation through a lesser-known Nano editor technique.

## User flag 🚩
I first ran `nmap` to look for open/vulnerable ports on the machine:

- Port 21 is open and allows for `anonymous ftp` login meaning we can login as the `anonymous` user without needing a password.
  - Within the `ftp` server there is a file called `note_to_jake.txt` which could be something to look at
- Port 22 is open for `ssh` remote login
- Port 80 is open which is hosting a `Apache` web server meaning this machine is hosting a website that we can take a look at

